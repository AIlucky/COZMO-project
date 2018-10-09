# COZMO-project

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
Pic_Path = "C:/Users/AIlucky/Android/cozmo_sdk_examples_1.4.4/tutorials/02_cozmo_face/New folder/bla.png" #EDIT Picture address HERE

########################################################################################################################

#FACE DETECTION#

def face_detect(imagePath):#Nutt Chairatana 61011300
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


#TAKING PICTURE"#Vorachat Somsuay 61011359


def cozmo_face_camera(robot: cozmo.robot.Robot):
    '''Continuously display Cozmo's camera feed back on his face.'''
    robot.camera.image_stream_enabled = True
    robot.camera.color_image_enabled = True

    print("Press CTRL-C to quit")

    while True:
        duration_s = 0.1  # time to display each camera frame on Cozmo's face

        latest_image = robot.world.latest_image

        if latest_image is not None:
            im = np.array(latest_image.raw_image)
            
            img = Image.fromarray(im, 'RGB')
            img.save(Pic_Path, 'png')
            img.show()

            num = face_detect(Pic_Path)
            break
            

        time.sleep(duration_s)

#################################################################################################################################

#EXECUTING COMMAND FOR TAKING PICTURE"
cozmo.robot.Robot.drive_off_charger_on_connect = False  # Cozmo can stay on his charger for this example
cozmo.run_program(cozmo_face_camera)

######################################################################################################################################

#LINE NOTIFICATION"

def _lineNotify(file, payload):
    import requests
    url = 'https://notify-api.line.me/api/notify'
    token = line_token	
    headers = {'Authorization':'Bearer '+token}
    return requests.post(url, headers=headers , files=file, data = payload)

def notifyPicture(url):
    
    import os.path
    
    if url and os.path.isfile(url):
        files = {"imageFile": open(url, "rb")}
        
    if face_detect(url) == 1:
        payload = {'message': "There is " + str(face_detect(url)) + " student in the classroom"}
    elif face_detect(url) == 0:
        payload = {'message': "There is no student in the classroom"}
    else:
        payload = {'message': "There are " + str(face_detect(url)) + " students in the classroom"}

    return _lineNotify(files, payload)

path = Pic_Path

#######################################################################################################################################

#EXECUTING COMMAND FOR LINE NOTIFICATION"
#print(_lineNotify(None, {'message':"123"}))
#to add additional message replace the string "123" with the message you want to put in
print(notifyPicture(path))

