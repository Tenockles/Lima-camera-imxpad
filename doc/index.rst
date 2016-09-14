.. _camera-imxpad:

imXPAD
------

.. image:: imxpad.jpg
   :scale: 10 %

Introduction
````````````

The imXpad detectors benefit of hybrid pixel technology, which leads to major advantages compared to the other technologies. These advantages are mainly provided by direct photon conversion and real time electronic analysis of X-ray photons. This allows for direct photon counting and energy selection. 

XPAD detectors key features compared to CCDs and CMOS pixels detectors are: 
- Noise suppression
- Energy selection
- Almost infinite dynamic range
- High Quantum Efficiency (DQE(0) ~100%, dose reduction)
- Ultra fast electronic shutter (10 ns)
- Frame rate > 500 Hz

Prerequisite
````````````
In order to operate the imXpad detector, the USB-server or the PCI-server must be running in the computer attached to the detector.

Installation & Module configuration
````````````````````````````````````
-  follow first the steps for the linux installation :ref:`linux_installation`

The minimum configuration file is *config.inc*:

.. code-block:: sh

  COMPILE_CORE=1
  COMPILE_SIMULATOR=0
  COMPILE_SPS_IMAGE=1
  COMPILE_IMXPAD=1
  COMPILE_CONFIG=1
  COMPILE_GLDISPLAY=0
  LINK_STRICT_VERSION=0
  export COMPILE_CORE COMPILE_SPS_IMAGE COMPILE_SIMULATOR \       
       COMPILE_IMXPAD \
       COMPILE_GLDISPLAY \
       LINK_STRICT_VERSION
    
-  start the compilation :ref:`linux_compilation`

-  finally for the Tango server installation :ref:`tango_installation`

Initialisation and Capabilities
````````````````````````````````

Camera initialisation
......................


imXpad camera must be initialisated using 2 parameters:
	1) The IP adress where the USB or PCI server is running
	2) The port number use by the server to communicate.
	
Std capabilites
................

* HwDetInfo

  getCurrImageType/getDefImageType():

* HwSync: 

  get/setTrigMode(): the only supported mode are IntTrig, ExtGate, ExtTrigMult, ExtTrigSingle.

Refer to: http://imxpad.com/templates/SoftwareDocumentation/softwareDocumentation.html for a whole description of detector capabilities.

Optional capabilites
.....................

This plugin does not use optional hardware capabilities.

How to use
````````````
This is a python code example for a simple test:

.. code-block:: python

  from Lima import imXpad
  from Lima import Core
  import time

  # Setting XPAD camera (IP, port)
  cam = imXpad.Camera('localhost', 3456)

  HWI = imXpad.Interface(cam)
  CT = Core.CtControl(HWI)
  CTa = CT.acquisition()
  CTs = CT.saving()

  #To specify where images will be stored using EDF format
  CTs.setDirectory("./Images")
  CTs.setPrefix("id24_")
  CTs.setFormat(CTs.RAW)
  CTs.setSuffix(".bin")
  CTs.setSavingMode(CTs.AutoFrame)
  CTs.setOverwritePolicy(CTs.Overwrite)

  #To set acquisition parameters
  CTa.setAcqExpoTime(0.001) #1 ms exposure time.
  CTa.setAcqNbFrames(10) # 10 images.
  CTa.setLatencyTime(0.005) # 5 ms latency time between images.

  #To change acquisition mode
  cam.setAcquisitionMode(cam.XpadAcquisitionMode.Standard)

  #To set Triggers. Possibilities: Core.IntTrig, Core.ExtGate, Core.ExtTrigMult, Core.ExtTrigSingle. 
  CTa.setTriggerMode(Core.IntTrig)

  #To set Outputs.
  cam.setOutputSignalMode(cam.XpadOutputSignal.ExposureBusy)

  #ASYNCHRONOS acquisition
  CT.prepareAcq()
  CT.startAcq()

  #SYNCHRONOUS acquisition
  CT.prepareAcq()
  CT.startAcq()
  cam.waitAcqEnd()

  #To abort current process
  #CT.stopAcq()

  #Load Calibration from file
  #cam.loadCalibrationFromFile("./S70.cfg")

  #Perform Calibrations 0-SLOW, 1-MEDIUM, 2-FAST
  #cam.calibrationOTN(0)
  #cam.calibrationOTNPulse(0)
  #cam.calibrationBEAM(1000000,60,0) # 1s->exposure time, 60->ITHL_MAX, 0->SLOW

  
