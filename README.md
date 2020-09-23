<div align="center">

## The New Java USB API


</div>

### Description

Learn the new art to control your USB with JAVA
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Soheil](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/soheil.md)
**Level**          |Advanced
**User Rating**    |4.0 (24 globes from 6 users)
**Compatibility**  |Java \(JDK 1\.2\)
**Category**       |[Coding Standards](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/coding-standards__2-87.md)
**World**          |[Java](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/java.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/soheil-the-new-java-usb-api__2-3965/archive/master.zip)





### Source Code

```
Introduction:
The last few years saw a huge boom in the software market with the computer reaching the doorstep of general public. With that, rose the demand for components and programs that would cater to this general segment of the market. The most important addition to the computer in the last few years is the USB port which can been seen on most of the recent computers. This facility greatly simplifies how third party devices or common devices like printers, scanners, digital cameras, and USB storage devices can interact with personal computers. Computers always had serial ports and parallel ports, which could be used to connect to third party devices but these, were limited in their capabilities as the ports were not dynamic and expandable. Most of the time any device connected to these ports required some installation or driver configuration and or a restart of the computer. Moreover these devices were not scalable, expandable or robust. Devices connected to USB devices are scalable, expandable, tree based, plug and play and, added to that most devices connected to the USB port do not need any installation and will start working just by plugging into the USB port. This makes it easier for manufacturers to create user friendly products and saves the customers the nightmares of installation.
Sun had seen a growth in the external device market and had proposed a Java communication API. The Java communication API provides access to RS232 serial ports and parallel ports from Java applications. The Java communication API though comprehensive enough to cover all communications to the computer was too generic for USB based devices. Moreover, since the USB specification is quite different from any serial port or parallel port specifications, a common specification for all these devices does not do justice to USB and USB based devices. For that reason IBM proposed a new JSR (JSR 80) using the Java Community Process for a new Java USB API. The intent of this article will be to educate readers on the important parts of the API and how it would be useful to programmers and vendors who make USB based products.
Architecture:
The following subsystems constitute the core of the Java USB Architecture.
OS Services This section defines the services required from the underlying operating system needed for implementing the USB specification
Device Model This section defines the main interface provided to the users of the USB Specification.
Events Model The USB events model defines a event model for USB pipes and devices.
USB Pipes Model These are logical pipes that help in communication between USB devices and their components.
USB Operations This section defines the kind of requests users can send to USB devices.
The core of the system is a USB driver. The USB driver uses the Device model to interact with the Operating system services. The USB driver also listens to the Events subsystem to listen for any events that are fires to the Device model or the USB pipes. The driver uses the USB pipes subsystem to propagate commands or requests to the USB operations subsystem.
The Components:
1. OS Services:
The heart of the OS Services subsystem is the UsbHostManager class. This class has the responsibility of being an interface to all the devices and hubs attached to the host system. The UsbHostManager uses the UsbServicesUtility class to load the properties from a jusb.properties file and creates an object of type UsbServices. The class that is created implements the UsbServices interface but in actuality it is the current implementation class for the UsbServices interface.
The UsbHostManager class is a singleton class that allows only one instance of itself per JVM. The UsbHostManager class is instantiated when the first client makes a request to it. Once the client gets an instance of the UsbHostManager class it registers for events through the UsbHostManager.getInstance().getUsbServices() .addUsbServicesListerner(listener) method. This notifies the client if any devices are attached to the system or when current devices are removed from the system. The UsbHostManager also provides access to the root USB hub which can be used to query about devices already connected to the hub. There could be multiple USB controllers attached to a host, the UsbHostManager provides information about the multiple USB controllers and provides for mechanisms to find devices attached to any of them. The client can iteratively search for devices on any controller using the UsbServices object.
2. Device Model:
According to the USB specification every USB device is a class that implements the UsbDevice interface and it is exposed as a UsbDevice object. Even USB hubs are treated as UsbDevice objects. The USB hubs is represented as a class that implements the UsbHub interface which in turn is extended from the UsbDevice interface. Every USB device may have multiple configurations associated with it and these configurations are represented by their respective UsbConfig interfaces. Each combination of USB device and UsbConfig interface exposes different functionality which is driven by the configuration parameters in the UsbConfig. Each individual combination has different end points associated with them, which can act as a source or synch of data for that USB device.
Typically clients use a combination of UsbDevice objects and UsbConfig objects to get access to a device. They then use the UsbConfig interface to get the corresponding UsbInterface for the device. The UsbInterface provides access to end points which are used for sending or receiving data.
3 Event Model:
The Event model of the USB specification is based on the JavaBeans event model. Since devices can be removed or attached dynamically, the model needs to have a mechanism to send and receive events. Each USB device has a set of UsbDeviceListeners that register themselves to receive notifications about any events. The device uses the UsbHostManager and UsbServices objects to add listeners to the host system. The USB events are delivered asynchronously to the device.
4 USB Pipes Model:
Pipes are the only means of communications between the host and USB devices. USB devices communicate through device end points, which in turn communicate through pipes. These pipes are not physical pipes for data transfer but are modeled as logical pipes. These logical pipes are objects that belong to a specific end point and exist as long as the device model exists.
For a pipe to be accessible the pipe must be active, pipes belonging to end points on an active interface are active while those on inactive interface settings are inactive. Apart from being active the Pipes need to be opened before they are used. To prepare a UsbPipe for communication, the UsbInterface that owns the UsbPipe needs to be claimed using the claim() method. If the UsbInterface is already in use by another client, an exception is thrown. The UsbPipe object can now be obtained using the getUsbPipe() method. Now the UsbPipe object can be used to open a pipe by calling the open() method. The pipe is now ready for any data transmission. A UspPipeException is thrown if the call fails.
The simplest way to transmit data through the pipes is using a byte array. The data that needs to be transmitted should be filled into the byte array if the direction of communication is out (From the host to the device) and vice versa if the communication is in. There are two types of communications possible, synchronous and asynchronous, which are defined in the UsbInterface. For synchronous communication the syncSubmit(byte []) method is used, while the asyncSubmit(byte []) method is used for asynchronous communication. The synchSubmit() method will block the control till the transmission is complete and return the number of bytes transferred or an exception in case of any problem. After the submission is complete all the UspPipe’s listeners will receive a data or error event depending upon the completion status of the operation. The asynchronous operation returns a SubmitResult object immediately after the subsystem accepts the submission. It does not wait till the transmission is completed but allows the client to use the SubmitResult object to track the submission. The asynchronous process has a method called waitUntilCompleted() which can be used to block control till the submission is completed.
The UspPipe’s endpoint’s direction determines if the pipe is being used for input or output. Output is when data is transferred from the host to the device and input is the reverse process. For input the data buffer is filled with data received from the device while for output the data is passed to the device. The specification does not have any maximum or minimum limitations on the size of the data. But the data size is limited to the UspPipe’s maximum packet size. However if the data exceeds the packet size it will be sent in segments.
5 Request and Standard Operations:
The Request and USB operations provide a mechanism of performing standard operations and vendor specific operations on the USB devices. The USB specification defines a few standard operations that all devices need to do, which is defined in the StandardOperations interface. The request results are available from the Request objects through get methods(). To cater to different request and the different data encoding mechanisms a RequestFactory Class is provided to create Requests.
USB Pipe Types
Control pipes are the simplest form of pipes used for communication. They are further divided into normal control pipes and default control pipes. Default control pipes are not directly accessible to the client but take requests from a client. Normal control pipes can be directly accessed by the client but require a specific data format where the first eight bytes of the data buffer is the setup packet. The first byte of the setup byte determines the direction and the rest of the seven bytes are used for the properties of the request.
Isochronous pipes are more complicated. The direction of data transfer in the pipe is determined by the end point. Isochronous transfers are time sensitive. When the host receives an isochronous transfer it identifies the frame number for the first frame. The specification does not provide the application the flexibility to specify the first frame, so the application needs to schedule its packets for the earliest possible frame, so each submission is treated as a single independent packet. Frame synchronization becomes the responsibility of the application, so it needs to provide enough packets to ensure frame synchronization.
USB Pipe State Model
A pipe exists in a two states: active or inactive. A pipe needs to be in the active state for any communication to happen. Within the Active state a pipe can be in three substates: idle, busy and error. When the pipe is opened it is in the idle state, once data starts to communicate it changes to the busy state and will remain there till completion of the transmission or till an error occurs. Once transmission is completed it gets back to the idle state and waits for further communication requests. The pipe cannot be closed during the busy state, but can be closed in the error state and busy state.
I/O Request Packets (IRP)
For simple communication a data buffer (byte []) can be used, but for complex communications I/O request packets are required. An IRP consists of a data buffer, communication policy information, and other meta data all packaged into a single object. IRPs give more control to the application on how the data is submitted; they can be programmed to resubmit themselves on completion. For more complicated submissions another type of IRP called the composite UsbIrp (UsbCompositeIrp) is used. They behave like normal UsbIrps but can encapsulate multiple IRPs which can be submitted uninterrupted. They also provide better optimization techniques to handle UsbIrps more efficiently.
Conclusion
The Java USB API is still in the public draft stage of the Java Community Process, so many aspects of the API are not very clear. The intent of the Java USB API is to provide a unified specification API that would allow developers and vendors to create drivers for their own application. This specification strives to provide a standard library for all platforms and is targeted for the J2ME and J2SE platforms. However we can hope that by the end of the JCP the specification team will provide a good API for USB access. It could throw open a wide range of devices and services into the market.
As of now www.SourceForge.net has an open source implementation of a Java USB specification called jUsb. The jUsb is an implementation specific to Linux and has no support for Windows based systems. jUsb also has a parallel windows version of the API being made but only a mention of it is provided on the website.
```

