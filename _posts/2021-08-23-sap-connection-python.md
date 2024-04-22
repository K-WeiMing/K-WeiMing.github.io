---
layout: post
title: Automating SAP with Python
subtitle: SAP Robotic Process Automation
cover-img: /assets/img/SAP-Python/sap_cover.jpeg
thumbnail-img: /assets/img/SAP-Python/sap_thumbnail.png
share-img: /assets/img/SAP-Python/sap_thumbnail.png
tags: [Robotic Process Automation, RPA, SAP, VBA]
readtime: True
---
# Introduction
Many companies around the world use SAP for various tasks, such as quality management, laboratory sample data, preventive maintenance scheduling, supply chain scheduling. However most companies do not give access to SAP Advanced Business Application Programming (ABAP) environment (Transaction code: SE38). Most users use SAP on a daily basis repeating the same tasks over and over again, such as data extraction and updating information in SAP which is repetitive and time consuming.

So how does the typical user overcome this limitation and digitalize workflows without having access to the ABAP environment? If your administrator did not disable the feature for scripting, you are in luck. (See screenshot below):

![SAP Settings For Scripting](/assets/img/SAP-Python/sap_settings_scripting.png)

## Why Python Instead of VB?
Majority of the code snippets on SAP GUI automation on the web involve the use of Visual Basic in conjunction with Microsoft Excel and not Python. Therefore it is difficult to find any good quality posts on how to utilize python in a SAP GUI environment.

The biggest question is as to why use Python when you can use Visual Basic? There are a couple of reasons that come to mind.

- Visual Basic is not as popular compared to python
- Not able to use Microsoft Excel while the script is running
- Utilize an output from SAP and chain it with another python script
- It is not possible to use multi-threading in Excel to execute SAP scripts

# Connecting SAP GUI

In order to perform any tasks in SAP using Python, we first have to connect to the SAP GUI. In order of sequence what is happening below:

1. Connect to the SAP GUI interface
2. Create a Scripting Object (`application`)
3. Loop through the open `application` objects and identify sessions that are on the main page. This is typically named as `SESSION_MANAGER`.
4. Return a session object which we will use to control SAP

*Note: Doing it this way prevents the script from overriding and exiting any current transactions you have and only looks for the default screen after logging in*
```python
# Install the required package
!pip install pywin32
```

```python
import win32com.client

def get_client():
  sap_gui_auto = win32com.client.GetObject("SAPGUI")
  if not type(sap_gui_auto) == win32com.client.CDispatch:
    return

  application = sap_gui_auto.GetScriptingEngine
  if not type(application) == win32com.client.CDispatch:
    sap_gui_auto = None
    return

  for conn in range(application.Children.Count):
    # Loop through the application and get the connection
    connection = application.Children(conn)

    for sess in range(connection.Children.Count):
      # Loop through each connection and return sessions that are on the main screen 'SESSION_MANAGER'
      session = connection.Children(sess)

      if session.Info.Transaction == 'SESSION_MANAGER':
        return session
      else:
        # Return None and break
        return

```

# Controlling SAP

Once we've connected to the SAP GUI, we want to access the transactions and interfaces. So how do we know what code to type without any documentation provided?

## Macro Recording

Within SAP, you can record your inputs by using the *'Script Recording and Playback'* feature within the client. If you've experienced using a *'Macro Recorder'* in Microsoft Excel, this will be similar. In order to access this feature, access the settings and record shown below.

![SAP Macro Record Settings](/assets/img/SAP-Python/sap_settings.png)

![SAP Record Macro](/assets/img/SAP-Python/sap_record_playback.png)

*Note: Make sure to fill in the 'Save To' for file_location. This is where we will view the recorded macro.*

Once you've completed the recording of the task you want to perform, go to the saved location of the SAP script file and open it in an editor such as Notepad++ to view the code. You should see something similar to what is shown below.

<u>Example</u>

For this example, we will be attempting to access the Batch Genealogy / Pedigree in SAP for a particular batch. For sensitivity reasons, the actual batch information will not be shown.

![SAP Main Screen Input](/assets/img/SAP-Python/sap_main_label.png)

![SAP Genealogy Input](/assets/img/SAP-Python/sap_genealogy_label.png)



```asp
If Not IsObject(application) Then
   Set SapGuiAuto  = GetObject("SAPGUI")
   Set application = SapGuiAuto.GetScriptingEngine
End If
If Not IsObject(connection) Then
   Set connection = application.Children(0)
End If
If Not IsObject(session) Then
   Set session    = connection.Children(0)
End If
If IsObject(WScript) Then
   WScript.ConnectObject session,     "on"
   WScript.ConnectObject application, "on"
End If

session.findById("wnd[0]").maximize
session.findById("wnd[0]/tbar[0]/okcd").text = "/nMB56"
session.findById("wnd[0]").sendVKey 0

session.findById("wnd[0]/usr/ctxtPA_MATNR").text = "material_number"
session.findById("wnd[0]/usr/ctxtPA_WERKS").text = "code"
session.findById("wnd[0]/usr/ctxtPA_CHARG").text = "bx_num"
session.findById("wnd[0]/usr/ctxtPA_CHARG").setFocus
session.findById("wnd[0]/usr/ctxtPA_CHARG").caretPosition = 6
session.findById("wnd[0]/tbar[1]/btn[7]").press

```

## Breaking Down The Code
The macro recorded generates the code in VBA. In order to only perform the required actions we need to understand what is going on with the generated code.

### 1. SAP Connection
With every Macro Recording, the generated code shows how a *.vb* script is connected to SAP. Do take note that the code below does not account for different SAP System ID and if your SAP window is on the default window, overriding whatever transaction you have on the screen if you choose to execute this code.

```vbnet
If Not IsObject(application) Then
   Set SapGuiAuto  = GetObject("SAPGUI")
   Set application = SapGuiAuto.GetScriptingEngine
End If
If Not IsObject(connection) Then
   Set connection = application.Children(0)
End If
If Not IsObject(session) Then
   Set session    = connection.Children(0)
End If
If IsObject(WScript) Then
   WScript.ConnectObject session,     "on"
   WScript.ConnectObject application, "on"
End If
```

### 2. Executing the Transaction
This section is where we want to convert the VB code into python.

What we see in the SAP script is documented in Visual Basic, so how do we translate this into our Python Script? In fact the syntax that the ScriptingEngine in the SAPGUI object does not require much modification from Visual Basic. An example below:


*Code in VB*:
```asp
session.findById("wnd[0]/tbar[0]/okcd").text = "/nMB56"
session.findById("wnd[0]").sendVKey 0

session.findById("wnd[0]/usr/ctxtPA_MATNR").text = "material_code"
session.findById("wnd[0]/usr/ctxtPA_WERKS").text = "code"
session.findById("wnd[0]/usr/ctxtPA_CHARG").text = "bx_num"
session.findById("wnd[0]/usr/ctxtPA_CHARG").setFocus
session.findById("wnd[0]/usr/ctxtPA_CHARG").caretPosition = 6
session.findById("wnd[0]/tbar[1]/btn[7]").press
```

*Code in Python:*
```python
session.findById("wnd[0]/tbar[0]/okcd").text = "/nMB56"

# Add parenthesis to '0'
session.findById("wnd[0]").sendVKey(0)

session.findById("wnd[0]/usr/ctxtPA_MATNR").text = "material_code"
session.findById("wnd[0]/usr/ctxtPA_WERKS").text = "code"
session.findById("wnd[0]/usr/ctxtPA_CHARG").text = "bx_num"
session.findById("wnd[0]/usr/ctxtPA_CHARG").caretPosition = 6

# Add parenthesis as .press() is a method.
session.findById("wnd[0]/tbar[1]/btn[7]").press()
```
We can see that there isn't a lot of changes that are required to convert them from VB to Python which is very useful when trying to quickly automate workflows.


# Putting It together

```python
import win32com.client

def get_client():
  sap_gui_auto = win32com.client.GetObject("SAPGUI")
  if not type(sap_gui_auto) == win32com.client.CDispatch:
    return

  application = sap_gui_auto.GetScriptingEngine
  if not type(application) == win32com.client.CDispatch:
    sap_gui_auto = None
    return

  for conn in range(application.Children.Count):
    # Loop through the application and get the connection
    connection = application.Children(conn)

    for sess in range(connection.Children.Count):
      # Loop through each connection and return sessions that are on the main screen 'SESSION_MANAGER'
      session = connection.Children(sess)

      if session.Info.Transaction == 'SESSION_MANAGER':
        return session
      else:
        # Return None and break
        return



def main():
  # Initialize an object to store the SAP session
  obj_sess = get_client()

  # Replace 'session' with initialized 'obj_sess'
  obj_sess.findById("wnd[0]/tbar[0]/okcd").text = "/nMB56"
  obj_sess.findById("wnd[0]").sendVKey(0)
  obj_sess.findById("wnd[0]/usr/ctxtPA_MATNR").text = "material_code"
  obj_sess.findById("wnd[0]/usr/ctxtPA_WERKS").text = "code"
  obj_sess.findById("wnd[0]/usr/ctxtPA_CHARG").text = "bx_num"
  obj_sess.findById("wnd[0]/usr/ctxtPA_CHARG").caretPosition = 6
  obj_sess.findById("wnd[0]/tbar[1]/btn[7]").press()

if __name__ == '__main__':
  main()

```



# Conclusion
Robotic Process Automation is something I have picked up on my own time to improve my workflow in my day-job. I hope this post gives some insight on controlling SAP using Python instead of the typical method of Visual Basic in Microsoft Excel.
