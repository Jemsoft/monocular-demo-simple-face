## Python Simple Face Detection Script

The app uses Monocular Python SDK and one of the Monocular API endpoints. We are going to use [Face Detection](?python#face-detection) and the Python [Pillow](https://pypi.python.org/pypi/Pillow/3.2.0)
module to create an app which detects faces in an image and plots their locations visually over the image.

### Setup

If you haven't already, create a Monocular app for our web app from the dashboard. If you don't know how you can find
 the instructions in the [documentation](?python#getting-started).

To begin producing Python software for Monocular, install either Python 2 (2.6 or 2.7) or Python 3 (3.3, 3.4 or 3.5).

Next you need to setup the Python package manager [pip](https://pip.pypa.io/en/stable/installing/) and install the Monocular SDK using:

`pip install monocular`

If you have previously installed, check for upgrades using:

`pip install --upgrade monocular`


### Development

Begin by importing the necessary Python modules, PIL (specifically the Pillow fork) is a dependency
of Monocular so it should already be installed. If it is not installed, install it using pip:

`pip install pillow`

For the purposes of this tutorial we only need the Image and ImageDraw modules in Pillow. Image provides us
with the ability to read an image and display it, and ImageDraw allow us to draw over the image.

```python
from PIL import Image, ImageDraw
import monocular
import json
import imghdr
```

Next, initialize Monocular with your `client_id` and `client_secret`, these can be found in one of your Apps on the Monocular dashboard.
Initializing Monocular tells the SDK what credentials to use when communicating with Monocular.
```
# Fill in your own App ID and secret here:
monocular.initialize({
    'client_id':'jemsonsAppId',
    'client_secret':'jemsonsAppSecret'
})
```
Now we need to get a file from the computer, we will do this by creating a loop which will ask the user to type in the path to a file and will only stop once we get a valid file path. Here we use the `imghdr` module to check that the image type is compatible with Monocular.

```
valid_types = ['png', 'jpeg', 'bmp']

invalid = True
while invalid:
    try:
        # Get the path from user input
        path = raw_input('Enter image path:')
        # Validate Image
        if imghdr.what(path) in valid_types:
            invalid = False
        else:
            print('Image must be png, jpg or bmp format')
    except IOError:
        print('File does not exist.')
```

At this point we are ready to begin interacting with Monocular. The [face detection](?python#face-detection) endpoint responds with JSON data containing the top left and bottom right points of each face. If we set the `landmarks` paramter to `True`  it will include a list of coordinates which represent the location of the 86 landmarks of each face detected in the image.
```
response = monocular.face_detection({'landmarks':True, 'image':path})
```
Finally we need to draw the image so that we can plot our response onto it. 
```
img = Image.open(path)
draw = ImageDraw.Draw(img)
```
We then iterate over every box in the response, drawing a rectangle from the top left to bottom right points. For each bounding box we iterate over the landmarks and plot each point. In this example we plotted each point 5 times, one for the center and one above, below and to the left and right by one pixel to give us a more pronounced point.

```
for box in response:
    draw.rectangle([(box['left'], box['top']), (box['right'], box['bottom'])], outline='#00ff00');

    i = 0;
    for landmark in box['landmarks']:
        # Draw large point
        draw.point([landmark[0]+1, landmark[1]], fill='#00ffff')
        draw.point([landmark[0]-1, landmark[1]], fill='#00ffff')
        draw.point([landmark[0], landmark[1]+1], fill='#00ffff')
        draw.point([landmark[0], landmark[1]-1], fill='#00ffff')
        draw.point([landmark[0], landmark[1]], fill='#00ff00')
        i = i + 1
```

The face detection app is now completed!
You can find the source code [here](https://github.com/Jemsoft/monocular-demo-simple-face).
