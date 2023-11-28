import time
import math
import smbus2
import pyrebase
import RPi.GPIO as GPIO
import Adafruit_DHT
import Adafruit_ADS1x15
from w1thermsensor import W1ThermSensor
import drivers
import pyrebase
import random
import max30100



# Initialize LCD display
display = drivers.Lcd()

# Constants for DHT11 sensor
DHT_SENSOR = Adafruit_DHT.DHT11
DHT_PIN = 17

# Constants for ADS1115
ADS1115_ADDRESS = 0x48
adc = Adafruit_ADS1x15.ADS1115(address=ADS1115_ADDRESS)

# Constants for GY-61 accelerometer
GY61_ACCEL_X_CHANNEL = 1
GY61_ACCEL_Y_CHANNEL = 2


mx30 = max30100.MAX30100()
mx30.enable_spo2()


config = {
  "apiKey": "a0AUHUABrA8XfUGPiwuyRTbc3wQM8pOYfMET2oT------", #change it 
  "authDomain": "aaaaanimal-monitoring-a0151.firebaseapp.com",    # change it
  "databaseURL": "https://aaaaanimal-monitoring-a0151-default-rtdb.firebaseio.com", #change it
  "storageBucket": "aaaanimal-monitoring-a0151.appspot.com"  #change it
}

firebase = pyrebase.initialize_app(config)

db = firebase.database()


# Calculate angles based on accelerometer data
def calculate_angles(accel_x, accel_y):
    angle_x = math.degrees(math.atan2(accel_y, math.sqrt(accel_x ** 2)))
    angle_y = math.degrees(math.atan2(accel_x, math.sqrt(accel_y ** 2)))
    return angle_x, angle_y

bus = smbus2.SMBus(1)

display.lcd_clear()

while True:
    # Read analog data from GY-61 accelerometer
    accel_x = adc.read_adc(GY61_ACCEL_X_CHANNEL)
    accel_y = adc.read_adc(GY61_ACCEL_Y_CHANNEL)

    # Calculate angles
    angle_x, angle_y = calculate_angles(accel_x, accel_y)
    # Convert values to integers
    angle_x_int = int(angle_x)
    angle_y_int = int(angle_y)

    mx30.read_sensor()
    mx30.ir, mx30.red
    hb = int(mx30.ir / 100)
    spo2 = int(mx30.red / 100)
    
    if mx30.ir != mx30.buffer_ir :
        print("Pulse:",hb);
    if mx30.red != mx30.buffer_red:
        print("SPO2:",spo2);
    
    # Read temperature from W1ThermSensor
    sensor = W1ThermSensor()
    temperature_celsius = sensor.get_temperature()

    # Print sensor values
    print(f"Temperature: {temperature_celsius:.2f} °C")
    print(f"Angle X: {angle_x:.2f} degrees")
    print(f"Angle Y: {angle_y:.2f} degrees")



    # Read humidity and temperature from DHT11 sensor
    humidity, temperature = Adafruit_DHT.read(DHT_SENSOR, DHT_PIN)
    if humidity is not None and temperature is not None:
        print(f"Temp={temperature:.1f}C Humidity={humidity:.1f}%")
        t = int(temperature)
        h = int(humidity)
    else:
        print("Sensor failure. Check wiring.")

    # Update LCD display with humidity and temperature
        # Update LCD display
    display.lcd_clear()
    display.lcd_display_string(f"ANMIAL BODY TEMP", 1)
    display.lcd_display_string(f"      {temperature_celsius:.1f}C  ", 2)
    time.sleep(0.5)
    display.lcd_clear()
    display.lcd_display_string(f"HUMIDITY: {t:.1f}%", 1)
    display.lcd_display_string(f"HUMIDITY: {h:.1f}%", 2)
    time.sleep(0.5)
    display.lcd_clear()
        
    display.lcd_display_string(f"ANGLE X: {angle_x:.1f} deg", 1)
    display.lcd_display_string(f"ANGLE Y: {angle_y:.1f} deg", 2)

    time.sleep(0.5)
    display.lcd_clear()
        
    display.lcd_display_string(f"Heart Beat: {hb:.1f} deg", 1)
    display.lcd_display_string(f"Spo2: {spo2:.1f} deg", 2)


    time.sleep(0.5)
    display.lcd_clear()
    display.lcd_display_string("  DATA  SEND ", 1)
    display.lcd_display_string("   FIREBASE ", 1)
    

    #-------send to firebase------------------
    data = {
        "Body Temperature":temperature_celsius,
        "Temperature": t,
        "Humidity": h,
        "Pulse": hb ,
        "Spo2": spo2,  
        "x": angle_x_int,
        "y": angle_y_int
    }

    print(data)

    db.child("Animalmonitoring").update(data)

    
    time.sleep(1)




