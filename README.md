%gui tk

import cv2
import tkinter as tk
from threading import Thread
from time import sleep

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")
eye_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_eye.xml")

root = tk.Tk()
root.title("Blink-Controlled Font Size")
font_size = 20
label = tk.Label(root, text="Blink to change font size!", font=("Helvetica", font_size))
label.pack(padx=20, pady=20)

blink_counter = 0

def update_font():
    global font_size
    font_size += 2
    if font_size > 40:
        font_size = 20
    label.config(font=("Helvetica", font_size))

def detect_blink():
    global blink_counter
    cap = cv2.VideoCapture(0)
    while True:
        ret, frame = cap.read()
        if not ret:
            break
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        faces = face_cascade.detectMultiScale(gray, 1.3, 5)
        for (x, y, w, h) in faces:
            roi_gray = gray[y:y+h, x:x+w]
            eyes = eye_cascade.detectMultiScale(roi_gray)
            if len(eyes) < 2:
                blink_counter += 1
                print("Blink detected!")
                update_font()
                sleep(1)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    cap.release()
    cv2.destroyAllWindows()

Thread(target=detect_blink, daemon=True).start()
root.mainloop()
