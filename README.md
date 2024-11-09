# PLANT-DISEASE-DETECTION
import cv2
import numpy as np
import requests
from datetime import datetime

# Set the HSV color ranges for healthy and diseased leaves
healthy_lower = np.array([30, 30, 30])
healthy_upper = np.array([90, 255, 255])

diseased_lower = np.array([0, 50, 50])
diseased_upper = np.array([30, 255, 255])

# Replace 'YOUR_BOT_TOKEN' with your Telegram bot token
TELEGRAM_BOT_TOKEN = '6753763119:AAHyyV_mrmCWC8GJDg2EE1sdr2zvZNsVz7s'
# Replace 'YOUR_CHAT_ID' with the chat ID where you want to send the images
TELEGRAM_CHAT_ID = '1430362856'

def classify_leaf(frame):
    # Convert the frame to the HSV color space
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    # Create masks for healthy and diseased leaves
    healthy_mask = cv2.inRange(hsv, healthy_lower, healthy_upper)
    diseased_mask = cv2.inRange(hsv, diseased_lower, diseased_upper)

    # Count the number of white pixels in each mask
    healthy_pixel_count = cv2.countNonZero(healthy_mask)
    diseased_pixel_count = cv2.countNonZero(diseased_mask)

    # Determine the category based on pixel counts
    if healthy_pixel_count > diseased_pixel_count:
        return "Healthy Leaf"
    else:
        return "Diseased Leaf"

def capture_and_send_image():
    # Create a VideoCapture object
    cap = cv2.VideoCapture(0)
    
    # Check if the camera is opened successfully
    if not cap.isOpened():
        print("Error: Could not open camera.")
        return

    while True:
        # Capture a frame
        ret, frame = cap.read()

        # Classify the leaf based on color
        result = classify_leaf(frame)

        # Display the result
        cv2.putText(frame, result, (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
        cv2.imshow('Plant Leaf Health Detection', frame)

        # Save the captured image if diseased
        if result == "Diseased Leaf":
            image_path = f'/home/pi/Pictures/diseased_plant{datetime.now().strftime("%Y%m%d%H%M%S")}.jpg'
            cv2.imwrite(image_path, frame)
            send_image_to_telegram(image_path)

        # Break the loop when 'q' is pressed
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    # Release the camera and close all windows
    cap.release()
    cv2.destroyAllWindows()

def send_image_to_telegram(image_path):
    try:
        # Telegram bot API URL for sending photos
        url = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendPhoto'
        
        # Prepare the photo data
        files = {'photo': open(image_path, 'rb')}
        data = {'chat_id': TELEGRAM_CHAT_ID}

        # Send the image to the specified chat ID
        response = requests.post(url, files=files, data=data)
        response.raise_for_status()  # Raise an error for unsuccessful responses

        print("Image sent successfully to Telegram!")

    except Exception as e:
        print(f"Error sending image to Telegram: {e}")

if _name_ == "_main_":
    # Capture and send the image
    capture_and_send_image()
