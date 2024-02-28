# Automatic Translation Lab

Estimated Completion Time: 5 minutes

In this lab you will:
- interact with an end-to-end radio frequency logging system
- learn about integrations of data science and infrastructure to radio frequency
- learn about WiFi HaLow, an alternate radio communication protocol for transferring data

## Architecture

```mermaid
flowchart LR
    Handset -- NBFM --> SDR4Space --- Whisper.cpp -- MQTT over HaLow --> Kibana
```

Above you will see the general architecture that you will be interacting with today. The premise of this setup is automating the translation of radio communications for situational awareness at an operations center.

### Handset

The first aspect of this architecture addresses the signal that we want to capture. There are many signals traversing the electromagnetic spectrum, so it is important to be specific here. It is rare that you will find one tool that is able to capture all of the protocols. Today, we will be focusing on narrow band frequency modulation (NBFM). NBFM is an analog modulation scheme that encodes voice data with variations in the frequency of a signal. A NBFM receiver translates these variations back into voice in a process called demodulation. You can learn more about analog and digital modulation in a great beginner-friendly series "Field Expedient SDR" ([ref 1](#references)).

### SDR4Space

SDR4Space is our open source demodulation application ([ref 2](#references)). This implementation is limited in that we must configure the receiver for the frequency on which our handset is transmitting, in our case 435.05 MHz. This has limited practical applications because handsets can be reprogrammed for a range of frequencies, but advanced users will be able to change the SDR4Space application to scan over a range of frequencies in order to find and decode the NBFM information of interest.

### Whisper.cpp

Once SDR4Space detects a NBFM transmission that exceeds a power threshold, it passes the electromagnetic spectrum capture to an AI translation model, whisper.cpp ([ref 3](#references)). Electromagnetic spectrum captures are typically stored in in-phase & quadrature format, best known as "IQ format". For beginners, it is best to think about this as the radio frequency analog to an executable file on a computer. If you want to learn more about IQ data, I suggest you explore PySDR ([ref 4](#references)), IQEngine ([ref 5](#references)), and the SigMF standard ([ref 6](#references)). PySDR is a fantastic getting-started guide to processing IQ information using Python, IQEngine is an AI IQ data recognition engine that is created by the author of PySDR, and SigMF is an IQ file format standard that helps users understand the type of information that is contained within an IQ file.

The underlying training model of Whisper.cpp is not open source, but its weights have been provided open source. This program can be loaded with different translation model sizes; larger models may be more accurate translation wise but they will take longer to run. We can also load models that are able to translate a variety of languages in addition to English. For a full listing of features, it is best to review Whisper.cpp's GitHub repository listed in the references.

### MQTT & HaLow

After Whisper.cpp finishes translating the IQ capture, it sends the translated text strings to a preconfigured MQTT server. MQTT is a networking protocol that is more efficient for embedded systems.

The connectivity to the MQTT server is provided via a non-standard protocol: WiFi HaLow 802.11ah ([ref 8](#references)). HaLow operates in the US 900 MHz industrial scientific medicine (ISM) band and uses a very similar protocol to WiFi 802.11 that is used in the 2.4 and 5 GHz bands. Using this technology on a lower frequency band means that we can extend the range between two nodes at a sacrifice in data rate. Furthermore, the Alfa HaLow-U device employs 802.11s ([ref 9](#references)), a mesh networking standard that enables each node to relay information. This leads to a network that can scale its physical footprint much further than traditional WiFi.

An added benefit of the Alfa HaLow-U is that they employ the RNDIS protocl which allows them to act as an ethernet adapter over USB; therefore, we can simply plug the devices into our laptop and receive an IP address on the HaLow mesh.

### Kibana

The final stage in the architecture enables a single operator to have a condensed view of the vast amount of IQ data that the distributed mesh collects. The ELK Stack (Elastic, Logstash, Kibana) is leveraged to provide this service. Operators can configure dashboards in numerous ways, but the simplest dashboard that we have configured today shows a word cloud of all of the NBFM translated voice. This word cloud shows the most frequent words that were spoken over the air.

## Hardware

Handset: Baofeng UV-5R ([ref 11](#references)) transmits in plain text NBFM. SDR4Space can only demodulate plain text, it is not capable of decoding encrypted voice.

HaLow mesh: Alfa HaLow-U ([ref 12](#references)) plug-and-play device that enables connectivity to a HaLow mesh. 

Edge RF Sensor: WarDragon Kit ([ref 13](#references)) pre-configured edge sensor that comes with SDR4Space, Whisper.cpp, and all software contained within DragonOS ([ref 14](#references))

MQTT and Kibana Server: Any laptop, desktop, or server unit capable of running MQTT and Kibana.

## Usage

1. Connect the Alfa HaLow-U to the laptop. Run `ip a` to confirm that you have picked up an IP address  on the HaLow mesh.
2. Pick up the handset and speak a brief sentence into the mic. You should immediately start to see that SDR4Space detected the NBFM tranmission and begins to decode.
3. It will take a minute or so, but whisper.cpp is running on the backend to translate the demodulated NBFM communications and send to the Kibana server.
4. Once the translation is complete, you will see it appear in the terminal and then it will populate in the Kibana word cloud. 

## References
1. Field Expedient Software Defined Radio
2. SDR4Space GitHub
3. Whisper.cpp
4. PySDR
5. IQEngine
6. SigMF
7. MQTT reference
8. HaLow reference 802.11ah
9. 802.11s reference
10. Kibana reference
11. Baofeng UV-5R purchase link
12. HaLow-U purchase link: ttps://www.alfa.com.tw/products/halow-u?variant=39467228758088
13. WarDragon Purchase Link: https://cemaxecuter.com/?product=wardragon-w-case
14. DragonOS reference