
# Google Summer of Code 2022 "Aztec Glyphs" Report

Improving the Visual Recognition of Aztec Hieroglyphs (Decipherment Tool) @ Red Hen Lab - GSoC 2022

- Here is my blog project for [GSoC 2022 ](https://lisardop.github.io/)

- Code was deployed and running in University of Oregon server as [AztecGlyphRecognition URL](https://aztecglyphrecognition.wired-humanities.org/)

- Another functional mirror URL is provided under Heroku hosting [here](https://aztecglyphrecognition.herokuapp.com)

- Listed via Internet Archive - Wayback Machine [snapshot](https://web.archive.org/web/20220903185244/https://aztecglyphrecognition.herokuapp.com/)

- Dataset was expanded with new additions from 'Matricula de Tributos' Codex. A copy of them is available [here](https://www.dropbox.com/sh/q0ld6ir0r2n2pn7/AAAjLrmcFfLra2mOe4tE7EZRa?dl=0)

# CHANGELOG

As a continuity of previous GSoC2021 project, there is a list of new implementations and changes:

- Default upload folder changed from app local 'static/uploads/' to '/tmp/aztecglyphrecongitiontempuploads/' (script creates this tmp subfolder if not exists)

- The app shows now the first 6 closed images related with user's uploaded image instead of just 5

- Results are printed in screen under 100 px instead 120 for fit with phone devices and some browsers

- Fixed: first most closed result is now readed from array(0) position, instead 1.

- Glyph info is obtained from first part of the filename split

- Each closed image is now also linked on-click to Visual Recognition of Aztec Hieroglyphs site's results

- Cosine distance is converted in percentil amount of similarity between user's image and closed image

- A proper match is >85% of similarity (very few times in lower ranks 80-75%)

- Glyph name and percentage are shown at image mouse-over

- Cropping glyphs with surrounded color material can give abnomal results due to full-color image comparison

# IMPLEMENTATION (localhost via 5000 port)

 Create a local virtual env (python3.9 and pip needed)
 
~~~
python3.9 -m ensurepip
python3.9 -m venv aztecglyphvenv
source ./aztecglyphvenv/bin/activate
~~~

>

- Install requirements (requirements.txt)

~~~
pip3.9 install flask flask-executor Werkzeug flask-socketio keras pillow python-socketio gunicorn==20.1.0 gevent-websocket eventlet==0.30.2 scipy tensorflow
~~~

>

- Run the script

~~~
export FLASK_APP=app
python3 -m flask run
~~~

>

- Check the app typing in your browser: http://localhost:5000

Enjoy!!


# License

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
