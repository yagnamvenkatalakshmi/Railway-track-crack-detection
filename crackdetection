import RPi.GPIO as GPIO
import serial
import time
import picamera
from PIL import Image
from PIL import ImageOps
from gps import *
from datetime import datetime

# Define GPIO pins
LDR_PIN = 17  # LDR sensor pin
LED_PIN = 27  # LED indicator pin

# Define motor control pins
enA = 18
in1 = 23
in2 = 24
enB = 25
in3 = 12
in4 = 16

# Set up GPIO
GPIO.setmode(GPIO.BCM)
GPIO.setup(LDR_PIN, GPIO.IN)
GPIO.setup(LED_PIN, GPIO.OUT)
GPIO.setup(enA, GPIO.OUT)
GPIO.setup(in1, GPIO.OUT)
GPIO.setup(in2, GPIO.OUT)
GPIO.setup(enB, GPIO.OUT)
GPIO.setup(in3, GPIO.OUT)
GPIO.setup(in4, GPIO.OUT)

# Set up PWM for motor control
motorA_pwm = GPIO.PWM(enA, 250)  # Frequency = 250Hz
motorB_pwm = GPIO.PWM(enB, 250)  # Frequency = 250Hz

# Start PWM with 0% duty cycle (motors stopped)
motorA_pwm.start(0)
motorB_pwm.start(0)

# Function to move forward
def forward():
    GPIO.output(in1, GPIO.LOW)
    GPIO.output(in2, GPIO.HIGH)
    GPIO.output(in3, GPIO.LOW)
    GPIO.output(in4, GPIO.HIGH)
    motorA_pwm.ChangeDutyCycle(12)  # Set motor A speed to 50%
    motorB_pwm.ChangeDutyCycle(12)  # Set motor B speed to 50%

# Function to stop
def stop():
    GPIO.output(in1, GPIO.LOW)
    GPIO.output(in2, GPIO.LOW)
    GPIO.output(in3, GPIO.LOW)
    GPIO.output(in4, GPIO.LOW)
    motorA_pwm.ChangeDutyCycle(0)  # Set motor A speed to 0%
    motorB_pwm.ChangeDutyCycle(0)  # Set motor B speed to 0%

# Function to capture image and preprocess
def capture_and_preprocess_image(output_file):
    with picamera.PiCamera() as camera:
        camera.resolution = (1280, 720)  # Set the resolution (adjust as needed)
        camera.start_preview()
        time.sleep(2)  # Wait for the camera to warm up
        camera.capture(output_file)  # Capture image
        # Open captured image
        img = Image.open(output_file)
        # Convert to grayscale
        gray_img = img.convert('L')
        # Equalize histogram
        equalized_img = ImageOps.equalize(gray_img)
        # Save the preprocessed image
        equalized_img.save(output_file)

# Function to get GPS data
def get_gps_data(gpsd):
    nx = gpsd.next()
    if nx and nx['class'] == 'TPV':
        # Extract GPS info
        latitude = getattr(nx, 'lat', "Unknown")
        longitude = getattr(nx, 'lon', "Unknown")
        speed = getattr(nx, 'speed', "Unknown")
        # Print GPS info to the console
        dateTimeObj = datetime.now()
        info = "Time: " + str(dateTimeObj) + " Position: lon = " + str(longitude) + ", lat = " + str(latitude) + " Speed: " + str(speed * 3.6) + " km/h"
        print(info)
        return latitude, longitude
    else:
        print("Failed to retrieve GPS data")
        return None, None

# Main loop
GPIO.output(LED_PIN, GPIO.HIGH)  # Turn on LED
try:
    gpsd = gps(mode=WATCH_ENABLE | WATCH_NEWSTYLE)
    while True:
        ldr_value = GPIO.input(LDR_PIN)
        
        if ldr_value == 0:  # If LDR detects light (crack detected)
            print("Crack detected")
            stop()  # Stop the cart
            latitude, longitude = get_gps_data(gpsd)  # Get GPS coordinates
            capture_and_preprocess_image("crack_image.jpg")
            forward()
            
        else:
            print("No crack detected")
            forward()  # Continue moving forward
        
        time.sleep(0.2)  # Check LDR value every half second

except KeyboardInterrupt:
    GPIO.cleanup()
