#!/usr/bin/env python3

# Copyright (c) 2016 Anki, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License in the file LICENSE.txt or at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

'''Display Cozmo's camera feed back on his face (like a mirror)
'''
from PIL import Image
import sys
import time
import imageio
try:
    import numpy as np
except ImportError:
    sys.exit("Cannot import numpy: Do `pip3 install --user numpy` to install")

try:
    from PIL import Image
except ImportError:
    sys.exit("Cannot import from PIL: Do `pip3 install --user Pillow` to install")

import cozmo
import cv2

line_token = 'Rhep2qrLKNDeSypFVLql5CKqqyAwUdI6h3KklpCr8gn' #EDIT Line address HERE
Pic_Path = "C:/Users/AIlucky/Desktop/FaceDetect-master/000000.png" #EDIT Picture address HERE
Total_Student = 60 #EDIT Number of student in classroom
path = Pic_Path


########################################################################################################################

#FACE DETECTION#

def face_detect(imagePath):
    # Get user supplied values
    cascPath = "haarcascade_frontalface_default.xml"

    # Create the haar cascade
    faceCascade = cv2.CascadeClassifier(cascPath)

    # Read the image
    image = cv2.imread(imagePath)
    gray = cv2.cvtColor(image, cv2.CASCADE_SCALE_IMAGE)

    # Detect faces in the image
    faces = faceCascade.detectMultiScale(
        gray,
        scaleFactor=1.2,
        minNeighbors=5,
        minSize=(30, 30),
        flags = cv2.CASCADE_SCALE_IMAGE
    )

    print("Found {0} faces!".format(len(faces)))

    # Draw a rectangle around the faces
    for (x, y, w, h) in faces:
        cv2.rectangle(image, (x, y), (x+w, y+h), (0, 255, 0), 2)

    cv2.imshow("Faces found", image)

    return len(faces)


####################################################################################################################################


#TAKING PICTURE"

cap = cv2.VideoCapture(0)
def capture_pic(robot: cozmo.robot.Robot):
    while True:
        _, frame = cap.read()
        cv2.imshow('frame', frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            cv2.imwrite('000000.png',frame)
            break
    notifyPicture(path,robot)



#################################################################################################################################



######################################################################################################################################

#LINE NOTIFICATION"

def _lineNotify(file, payload):
    import requests
    url = 'https://notify-api.line.me/api/notify'
    token = line_token	
    headers = {'Authorization':'Bearer '+token}
    return requests.post(url, headers=headers , files=file, data = payload)

def notifyPicture(url,robot: cozmo.robot.Robot):
    
    import os.path
    
    if url and os.path.isfile(url):
        files = {"imageFile": open(url, "rb")}
        
    S = Total_Student
    Absent = S - face_detect(url)
    
    if face_detect(url) == 1:
        if Absent == 1:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: 1"}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and 1 student who's absent.").wait_for_completed()
        elif Absent > 1:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: " + str(Absent)}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and " + str(Absent) + " student who's absent.").wait_for_completed()
        else:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: " + str(Absent)}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and no student who's absent.").wait_for_completed()

    elif face_detect(url) == 0:
        payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: " + str(Absent)}
        robot.say_text("There is no student in the classroom.").wait_for_completed()
        
    else:
        if Absent == 1:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: 1"}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and 1 student who's absent.").wait_for_completed()
        elif Absent > 1:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: " + str(Absent)}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and " + str(Absent) + " student who's absent.").wait_for_completed()
        else:
            payload = {'message': "   Present: " + str(face_detect(url)) + "     Absent: " + str(Absent)}
            robot.say_text("There are " + str(face_detect(url)) + " student in the classroom and no student who's absent.").wait_for_completed()

    return _lineNotify(files, payload)


    notifyPicture(path,robot)
#######################################################################################################################################
#EXECUTING COMMAND FOR TAKING PICTURE"
cozmo.robot.Robot.drive_off_charger_on_connect = False  # Cozmo can stay on his charger for this example
cozmo.run_program(capture_pic)

#cozmo.run_program(cozmo_face_camera)
#EXECUTING COMMAND FOR LINE NOTIFICATION"
#print(_lineNotify(None, {'message':"123"}))
#to add additional message replace the string "123" with the message you want to put in
