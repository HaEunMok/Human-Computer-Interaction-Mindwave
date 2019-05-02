
# Mindwave
### Member
- 박규태, 문현기, 박은하, 목하은 

## Introduction

"Sleep Bebe"는 뇌파를 분석하여 아기의 수면 정도에 따른 알람을 해주는 시스템으로, 아기와 떨어져 있는 공간에서도 스마트폰앱과 스마트 전구 휴(Hue)로 아기가 잠에서 깼는 지 알려준다. 아기의 수면 정도에 따라 휴의 색이 바뀌고, 앱에서는 전구모형의 그림의 색이 바뀐다. 예를 들어, 아이의 잠이 깬 상태에서는 전구가 빨간색으로 빛나고 휴대폰에 알람이 온다. 아기의 수면도를 측정하고 언제 깰 지 예측할 뿐만 아니라, 아이의 수면도에 따른 통계를 추가함으로써 아기에게 관심이 많은 부모들에게 관련 정보를 제공하고자 하였다. 

<center><img src="/baby.jpg"></center>

## Planning
### 1. Background
- 아기가 자는 동안, 부모님들은 휴식을 취하거나 다른 일을 할 수 없음
- 아기가 잠에서 깼을 때, 부모님이 눈 앞에 보이지 않으면 불안감을 느낄 수 있음
- 청각장애인들은 아기의 울음소리를 들을 수 없음
### 2. Objectives: 3R
- Rest for mother: 아기가 잘 때, 부모님들이 휴식을 취하거나 잠시 다른 일을 할 수 있도록 함
- Relief of child: 아기가 깼을 때, 눈을 뜨면 부모님을 보며 안정감을 찾을 수 있도록 함
- Replace sound to light: 청각 장애인 부모님들에게 아기의 우는 소리를 시각적으로 알려줌
### 3. storyboard
![Human-Computer-Interaction-Mindwave](Storyboard.pdf)


### 4. 구성요소: 앱, 휴, EEG
EEG와 앱, 휴를 블루투스로 연동하여 EEG로 측정된 값을 스마트폰앱과 휴로 전송하는 방식으로 작동된다. 

#### 4-1 앱: 
자는 아기와 거리상 떨어져 있더라도 아기의 수면상태를 파악할 수 있도록 앱을 통해 알려준다. 본 프로젝트에서는 XD프로그램을 통해 앱의 프로토타입을 제작하였으나, planning단계에서는 실제 데이터를 볼 수 있는 앱 사용을 고려하였다. 앱의 용도는 컴퓨터 연결로 대체하였다. 
#### 4-2 휴: 
아이의 소리를 들을 수 없는 청각 장애인들도 아기가 깼다는 것을 인지할 수 있도록 스마트 전구를 사용하여 알려 줄예정이다. 
#### 4-3 EEG: 
EEG 측정은 본 project에서 mindwave mobile 기기를 사용하여 실험하였으나, 제품 planning 단계에서는 무선 EEG기기 사용을 고려하였다.  
## Prototype
### 1. paper prototype
![Human-Computer-Interaction-Mindwave](paper prototype.pdf)
### 2. low-prototype (APP)
1차 프로토타입은 User research를 바탕으로 제작되었다.

**User research** 
<center><img src="/concept feedback.png"></center>

**1차 프로토 타입**
[Human-Computer-Interaction-Mindwave](https://xd.adobe.com/view/e8a4776e-a197-4c6b-4b89-7b2da3243a68-9e95/)
<center><img src="/entire prototype.png"></center>

아래는 Usability Test의 feedback으로 이를 바탕으로 2차 프로토 타입을 제작하였다. 

**Usability test**
<center><img src="/prototype weakness.png"></center>

**2차 프로토타입**
[Human-Computer-Interaction-Mindwave](https://xd.adobe.com/view/e8a4776e-a197-4c6b-4b89-7b2da3243a68-9e95/)
<center><img src="/entire prototype2.png"></center>


### 3. setting
#### 1. Windows 10에 Pybluez 설치
1. Anaconda 설치 (Python 3.7)
2. 아래 사이트를 참고하여 PyBluez 설치: https://github.com/pybluez/pybluez/issues/180#issuecomment-448102727
	1) Download and run "Visual Studio Installer": https://www.visualstudio.com/pl/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15
	2) Install "Visual Studio Build Tools 2017", check "Visual C++ build tools" and "Universal Windows Platform build tools"
	3) git clone https://github.com/pybluez/pybluez
	4) cd pybluez
	5) python setup.py install

### 4. high-prototype(code)

>from phue import Bridge
from openpyxl import Workbook
import random
import csv
import time
import bluetooth
import textwrap
import struct
import collections

#MindwaveDataPointReader--------------------------
class MindwaveDataPointReader:
    def __init__(self, address=None):
        self._mindwaveMobileRawReader = MindwaveMobileRawReader(address=address)
        self._dataPointQueue = collections.deque()

    def start(self):
        self._mindwaveMobileRawReader.connectToMindWaveMobile()

    def isConnected(self):
        return self._mindwaveMobileRawReader.isConnected()

    def readNextDataPoint(self):
        if (not self._moreDataPointsInQueue()):
            self._putNextDataPointsInQueue()
        return self._getDataPointFromQueue()

    def _moreDataPointsInQueue(self):
        return len(self._dataPointQueue) > 0

    def _getDataPointFromQueue(self):
        return self._dataPointQueue.pop();

    def _putNextDataPointsInQueue(self):
        dataPoints = self._readDataPointsFromOnePacket()
        self._dataPointQueue.extend(dataPoints)

    def _readDataPointsFromOnePacket(self):
        self._goToStartOfNextPacket()
        payloadBytes, checkSum = self._readOnePacket()
        if (not self._checkSumIsOk(payloadBytes, checkSum)):
            print("checksum of packet was not correct, discarding packet...")
            return self._readDataPointsFromOnePacket();
        else:
            dataPoints = self._readDataPointsFromPayload(payloadBytes)
        self._mindwaveMobileRawReader.clearAlreadyReadBuffer()
        return dataPoints;

    def _goToStartOfNextPacket(self):
        while(True):
            byte = self._mindwaveMobileRawReader.getByte()
            if (byte == MindwaveMobileRawReader.START_OF_PACKET_BYTE):  # need two of these bytes at the start..
                byte = self._mindwaveMobileRawReader.getByte()
                if (byte == MindwaveMobileRawReader.START_OF_PACKET_BYTE):
                    # now at the start of the packet..
                    return;

    def _readOnePacket(self):
            payloadLength = self._readPayloadLength();
            payloadBytes, checkSum = self._readPacket(payloadLength);
            return payloadBytes, checkSum

    def _readPayloadLength(self):
        payloadLength = self._mindwaveMobileRawReader.getByte()
        return payloadLength

    def _readPacket(self, payloadLength):
        payloadBytes = self._mindwaveMobileRawReader.getBytes(payloadLength)
        checkSum = self._mindwaveMobileRawReader.getByte()
        return payloadBytes, checkSum

    def _checkSumIsOk(self, payloadBytes, checkSum):
        sumOfPayload = sum(payloadBytes)
        lastEightBits = sumOfPayload % 256
        invertedLastEightBits = self._computeOnesComplement(lastEightBits) #1's complement!
        return invertedLastEightBits == checkSum;

    def _computeOnesComplement(self, lastEightBits):
        return ~lastEightBits + 256

    def _readDataPointsFromPayload(self, payloadBytes):
        payloadParser = MindwavePacketPayloadParser(payloadBytes)
        return payloadParser.parseDataPoints();

#MindwaveDataPoints-------------------
class DataPoint:
    def __init__(self, dataValueBytes):
        self._dataValueBytes = dataValueBytes

class UnknownDataPoint(DataPoint):
    def __init__(self, dataValueBytes):
        DataPoint.__init__(self, dataValueBytes)
        self.unknownPoint = self._dataValueBytes[0]

    def __str__(self):
        retMsgString = "Unknown OpCode. Value: {}".format(self.unknownPoint)
        return retMsgString

class PoorSignalLevelDataPoint(DataPoint):
    def __init__(self, dataValueBytes):
        DataPoint.__init__(self, dataValueBytes)
        self.amountOfNoise = self._dataValueBytes[0];

    def headSetHasContactToSkin(self):
        return self.amountOfNoise < 200;

    def __str__(self):
        poorSignalLevelString = "Poor Signal Level: " + str(self.amountOfNoise)
        if (not self.headSetHasContactToSkin()):
            poorSignalLevelString += " - NO CONTACT TO SKIN"
        return poorSignalLevelString

class AttentionDataPoint(DataPoint):
    def __init__(self, _dataValueBytes):
        DataPoint.__init__(self, _dataValueBytes)
        self.attentionValue = self._dataValueBytes[0]

    def __str__(self):
        return "Attention Level: " + str(self.attentionValue)

class MeditationDataPoint(DataPoint):
    def __init__(self, _dataValueBytes):
        DataPoint.__init__(self, _dataValueBytes)
        self.meditationValue = self._dataValueBytes[0]

    def __str__(self):
        return "Meditation Level: " + str(self.meditationValue)

class BlinkDataPoint(DataPoint):
    def __init__(self, _dataValueBytes):
        DataPoint.__init__(self, _dataValueBytes)
        self.blinkValue = self._dataValueBytes[0]

    def __str__(self):
        return "Blink Level: " + str(self.blinkValue)

class RawDataPoint(DataPoint):
    def __init__(self, dataValueBytes):
        DataPoint.__init__(self, dataValueBytes)
        self.rawValue = self._readRawValue()

    def _readRawValue(self):
        firstByte = self._dataValueBytes[0]
        secondByte = self._dataValueBytes[1]
        # TODO(check if this is correct iwth soem more tests..
        # and see http://stackoverflow.com/questions/5994307/bitwise-operations-in-python
        rawValue = firstByte * 256 + secondByte;
        if rawValue >= 32768:
            rawValue -= 65536
        return rawValue # hope this is correct ;)

    def __str__(self):
        return "Raw Value: " + str(self.rawValue)

class EEGPowersDataPoint(DataPoint):
    def __init__(self, dataValueBytes):
        DataPoint.__init__(self, dataValueBytes)
        self._rememberEEGValues();

    def _rememberEEGValues(self):
        self.delta = self._convertToBigEndianInteger(self._dataValueBytes[0:3]);
        self.theta = self._convertToBigEndianInteger(self._dataValueBytes[3:6]);
        self.lowAlpha = self._convertToBigEndianInteger(self._dataValueBytes[6:9]);
        self.highAlpha = self._convertToBigEndianInteger(self._dataValueBytes[9:12]);
        self.lowBeta = self._convertToBigEndianInteger(self._dataValueBytes[12:15]);
        self.highBeta = self._convertToBigEndianInteger(self._dataValueBytes[15:18]);
        self.lowGamma = self._convertToBigEndianInteger(self._dataValueBytes[18:21]);
        self.midGamma = self._convertToBigEndianInteger(self._dataValueBytes[21:24]);


    def _convertToBigEndianInteger(self, threeBytes):
        # TODO(check if this is correct iwth soem more tests..
        # and see http://stackoverflow.com/questions/5994307/bitwise-operations-in-python
        # only use first 16 bits of second number, not rest inc ase number is negative, otherwise
        # python would take all 1s before this bit...
        # same with first number, only take first 8 bits...
        bigEndianInteger = (threeBytes[0] << 16) |\
         (((1 << 16) - 1) & (threeBytes[1] << 8)) |\
          ((1 << 8) - 1) & threeBytes[2]
        return bigEndianInteger

    def __str__(self):
        return """EEG Powers:
                delta: {self.delta}
                theta: {self.theta}
                lowAlpha: {self.lowAlpha}
                highAlpha: {self.highAlpha}
                lowBeta: {self.lowBeta}
                highBeta: {self.highBeta}
                lowGamma: {self.lowGamma}
                midGamma: {self.midGamma}
                """.format(self = self)


#MindwaveMobileRawReader--------------------

class MindwaveMobileRawReader:
    START_OF_PACKET_BYTE = 0xaa;
    def __init__(self, address=None):
        self._buffer = [];
        self._bufferPosition = 0;
        self._isConnected = False;
        self._mindwaveMobileAddress = address

    def connectToMindWaveMobile(self):
        # First discover mindwave mobile address, then connect.
        # Headset address of my headset was'9C:B7:0D:72:CD:02';
        # not sure if it really can be different?
        # now discovering address because of https://github.com/robintibor/python-mindwave-mobile/issues/4
        if (self._mindwaveMobileAddress is None):
            self._mindwaveMobileAddress = self._findMindwaveMobileAddress()
        if (self._mindwaveMobileAddress is not None):
            print ("Discovered Mindwave Mobile...")
            self._connectToAddress(self._mindwaveMobileAddress)
        else:
            self._printErrorDiscoveryMessage()

    def _findMindwaveMobileAddress(self):
        nearby_devices = bluetooth.discover_devices(lookup_names = True)
        for address, name in nearby_devices:
            if (name == "MindWave Mobile"):
                return address
        return None

    def _connectToAddress(self, mindwaveMobileAddress):
        self.mindwaveMobileSocket = bluetooth.BluetoothSocket(bluetooth.RFCOMM)
        while (not self._isConnected):
            try:
                self.mindwaveMobileSocket.connect(
                    (mindwaveMobileAddress, 1))
                self._isConnected = True
            except bluetooth.btcommon.BluetoothError as error:
                print("Could not connect: ", error, "; Retrying in 5s...")
                time.sleep(5)


    def isConnected(self):
        return self._isConnected

    def _printErrorDiscoveryMessage(self):
         print((textwrap.dedent("""\
                    Could not discover Mindwave Mobile. Please make sure the
                    Mindwave Mobile device is in pairing mode and your computer
                    has bluetooth enabled.""").replace("\n", " ")))

    def _readMoreBytesIntoBuffer(self, amountOfBytes):
        newBytes = self._readBytesFromMindwaveMobile(amountOfBytes)
        self._buffer += newBytes

    def _readBytesFromMindwaveMobile(self, amountOfBytes):
        missingBytes = amountOfBytes
        # receivedBytes = ""  #py2
        receivedBytes = b''   #py3

        # Sometimes the socket will not send all the requested bytes
        # on the first request, therefore a loop is necessary...
        while(missingBytes > 0):
            receivedBytes += self.mindwaveMobileSocket.recv(missingBytes)
            missingBytes = amountOfBytes - len(receivedBytes)
        return receivedBytes;

    def peekByte(self):
        self._ensureMoreBytesCanBeRead();
        return ord(self._buffer[self._bufferPosition])

    def getByte(self):
        self._ensureMoreBytesCanBeRead(100);
        return self._getNextByte();

    def  _ensureMoreBytesCanBeRead(self, amountOfBytes):
        if (self._bufferSize() <= self._bufferPosition + amountOfBytes):
            self._readMoreBytesIntoBuffer(amountOfBytes)

    def _getNextByte(self):
        # nextByte = ord(self._buffer[self._bufferPosition]) #py2
        nextByte = self._buffer[self._bufferPosition]   #py3
        self._bufferPosition += 1;
        return nextByte;

    def getBytes(self, amountOfBytes):
        self._ensureMoreBytesCanBeRead(amountOfBytes);
        return self._getNextBytes(amountOfBytes);

    def _getNextBytes(self, amountOfBytes):
        # nextBytes = list(map(ord, self._buffer[self._bufferPosition: self._bufferPosition + amountOfBytes])) #py2
        nextBytes = list(self._buffer[self._bufferPosition: self._bufferPosition + amountOfBytes]) #py3
        self._bufferPosition += amountOfBytes
        return nextBytes

    def clearAlreadyReadBuffer(self):
        self._buffer = self._buffer[self._bufferPosition : ]
        self._bufferPosition = 0;

    def _bufferSize(self):
        return len(self._buffer);

#payloadParser--------------------

EXTENDED_CODE_BYTE = 0x55

class MindwavePacketPayloadParser:

    def __init__(self, payloadBytes):
        self._payloadBytes = payloadBytes
        self._payloadIndex = 0

    def parseDataPoints(self):
        dataPoints = []
        while (not self._atEndOfPayloadBytes()):
            dataPoint = self._parseOneDataPoint()
            dataPoints.append(dataPoint)
        return dataPoints

    def _atEndOfPayloadBytes(self):
        return self._payloadIndex == len(self._payloadBytes)

    def _parseOneDataPoint(self):
        dataRowCode = self._extractDataRowCode();
        dataRowValueBytes = self._extractDataRowValueBytes(dataRowCode)
        return self._createDataPoint(dataRowCode, dataRowValueBytes)

    def _extractDataRowCode(self):
        return self._ignoreExtendedCodeBytesAndGetRowCode()

    def _ignoreExtendedCodeBytesAndGetRowCode(self):
        # EXTENDED_CODE_BYTES seem not to be used according to
        # http://wearcam.org/ece516/mindset_communications_protocol.pdf
        # (August 2012)
        # so we ignore them
        byte = self._getNextByte()
        while (byte == EXTENDED_CODE_BYTE):
            byte = self._getNextByte()
        dataRowCode = byte
        return dataRowCode

    def _getNextByte(self):
        nextByte = self._payloadBytes[self._payloadIndex]
        self._payloadIndex += 1
        return nextByte

    def _getNextBytes(self, amountOfBytes):
        nextBytes = self._payloadBytes[self._payloadIndex : self._payloadIndex + amountOfBytes]
        self._payloadIndex += amountOfBytes
        return nextBytes

    def _extractDataRowValueBytes(self, dataRowCode):
        lengthOfValueBytes = self._extractLengthOfValueBytes(dataRowCode)
        dataRowValueBytes = self._getNextBytes(lengthOfValueBytes)
        return dataRowValueBytes

    def _extractLengthOfValueBytes(self, dataRowCode):
        # If code is one of the mysterious initial code values
        # return before the extended code check
        if dataRowCode == 0xBA or dataRowCode == 0xBC:
            return 1

        dataRowHasLengthByte = dataRowCode > 0x7f
        if (dataRowHasLengthByte):
            return self._getNextByte()
        else:
            return 1

    def _createDataPoint(self, dataRowCode, dataRowValueBytes):
        if (dataRowCode == 0x02):
            return PoorSignalLevelDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0x04):
            return AttentionDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0x05):
            return MeditationDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0x16):
            return BlinkDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0x80):
            return RawDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0x83):
            return EEGPowersDataPoint(dataRowValueBytes)
        elif (dataRowCode == 0xba or dataRowCode == 0xbc):
            return UnknownDataPoint(dataRowValueBytes)
        else:
            assert False

#example-----------------------------------------------------
b = Bridge("192.168.0.19")
#b.connect()
b.set_light(5,'on', True)


write_wb = Workbook()
write_wb = write_wb.active


if __name__ == '__main__':
    mindwaveDataPointReader = MindwaveDataPointReader()
    mindwaveDataPointReader.start()
    if (mindwaveDataPointReader.isConnected()):
        #for i in range(1000):
        while(True):
            dataPoint = mindwaveDataPointReader.readNextDataPoint()
            if (not dataPoint.__class__ is RawDataPoint):
                #print(dir(dataPoint))
                if hasattr(dataPoint, 'meditationValue'):
                    print(dataPoint.meditationValue)
                    medVal = dataPoint.meditationValue
                    b.set_light(5,'bri', medVal)

                    
                if hasattr(dataPoint, 'attentionValue'):
                    print(dataPoint.attentionValue)
                    attVal = dataPoint.attentionValue
                    b.set_light(5,'bri', medVal)
                    csv_writer.writerow([medVal, ])


    else:
        print((textwrap.dedent("""\
            Exiting because the program could not connect
            to the Mindwave Mobile device.""").replace("\n", " ")))


f.close()
