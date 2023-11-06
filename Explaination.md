### DJI Tello Drone Face Tracking and Control

This code demonstrates how to use the Tello SDK to control a DJI Tello drone based on face tracking using OpenCV.

#### Dependencies

- djitellopy
- numpy
- cv2

#### Code Explanation

1. **Import Libraries:**
   ```python
   from djitellopy import Tello
   import numpy as np
   import cv2
   import time

2. **Define Drone Speed:**
    ```python
   speed = 25

3. **Connect to Tello Drone:**
    ```python
   tello = Tello()
    tello.connect()
    tello.set_speed(speed)
    tello.streamon()

4. **Takeoff and Initialize Face Detection:**
    ```python
   tello.takeoff()
    time.sleep(2)
    tello.send_rc_control(0, 0, 30, 0)
    time.sleep(1)

    frame_read = tello.get_frame_read()

    face_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')

5. **Meaning of: face_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')**
    ```python
   The face_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml') statement in Python code is used to load a pre-trained face detector from an XML file named haarcascade_frontalface_default.xml. This XML file contains information about the Haar-like features that are used to detect faces in images. The cv2.CascadeClassifier() function takes the path to the XML file as an argument and returns a CascadeClassifier object.
   
6. **Main Loop:**
   ```python
   while running:
    key = cv2.waitKey(1)

    if key == ord('q'):
        cv2.destroyAllWindows()
        break

    if frame_read.stopped:
        break

    # Convert frame to RGB format and flip it horizontally
    frame = cv2.cvtColor(frame_read.frame, cv2.COLOR_BGR2RGB)
    frame = np.fliplr(frame)

    # Detect faces using OpenCV's face cascade classifier
    faces = face_cascade.detectMultiScale(frame, 1.5, 2)

    # If no faces detected, don't move the drone
    if len(faces) == 0:
        velocity_fb = 0
        velocity_lr = 0
        velocity_ud = 0
        velocity_yaw = 0

    # Process each detected face
    for (x, y, w, h) in faces:
        # Draw a rectangle and circle around the detected face
        cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)
        cv2.circle(frame, (x + w // 2, y + h // 2), 5, (0, 0, 255))

        # Calculate the center of the detected face and the center of the frame
        face_middle = (x + w // 2, y + h // 2)
        frame_middle = (frame.shape[1] // 2, frame.shape[0] // 2)

        # Calculate the displacement vector between the face center and the frame center
        z_axis_displacement = (200 - w, 200 - h)
        displacement_vector = np.array(
            [frame_middle[0] - face_middle[0], z_axis_displacement[0], frame_middle[1] - face_middle[1], 0])

        # Draw a line connecting the face center and the adjusted face center
        cv2.line(frame, (x + w // 2, y + h // 2),
                 (x + w // 2 + displacement_vector[0], y + h // 2 + displacement_vector[2]), (255, 255, 255), 2)

        # Adjust the drone's movement based on the displacement vector
        if not abs(displacement_vector[0]) < 200:
            velocity_lr = speed * int(displacement_vector[0] / abs(displacement_vector[0]))
        else:
            velocity_lr = 0

        if not abs(displacement_vector[1]) < 40:
            velocity_fb = speed * int(displacement_vector[1] / abs(displacement_vector[1]))
        else:
            velocity_fb = 0

        if not abs(displacement_vector[2]) < 200:
            velocity_ud = speed * int(displacement_vector[2] / abs(displacement_vector[2]))
        else:
            velocity_ud = 0

        break  # Only process the first detected face

    # Send the calculated velocity commands to the drone
   

Check the main loop explanation in [MainLoop.md](MainLoop.md)
