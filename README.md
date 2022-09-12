
# Google Summer of Code 2022 "Aztec Glyphs" Report

Improving the Visual Recognition of Aztec Hieroglyphs (Decipherment Tool) @ Red Hen Lab - GSoC 2022

- Here is my blog project for [GSoC 2022](https://lisardop.github.io/) and a post with all progress I did in [Code Period](https://lisardop.github.io/2022-06-14-weeks-2022/)

- Code was deployed and running in University of Oregon server as [AztecGlyphRecognition URL](https://aztecglyphrecognition.wired-humanities.org/)

- Another functional mirror URL is provided under Heroku hosting [here](https://aztecglyphrecognition.herokuapp.com)

- Listed via Internet Archive - Wayback Machine [snapshot](https://web.archive.org/web/20220903185244/https://aztecglyphrecognition.herokuapp.com/)

- Dataset was expanded with new additions from 'Matricula de Tributos' Codex. A copy of them is available [here](https://www.dropbox.com/sh/q0ld6ir0r2n2pn7/AAAjLrmcFfLra2mOe4tE7EZRa?dl=0)

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
python3.9 -m flask run
~~~

>

- Check the app typing in your browser: http://localhost:5000

Enjoy!!

# IMPLEMENTATION (deploy server, CentOs7)

- Follow same steps as in "localhost" description

- Install and config a Nginx server

- Install "supervisor" (it will keep our python app up and alive)

- We use gunicorn + eventlet from our virtual enviroment 'aztecglyphvenv'. Script runs on 0.0.0.0:5000 and gunicorn on 0.0.0.0:8000 by default, Nginx acts as reverse proxy 443 (outside) 8000 (inside). Files needed to configure:

## /etc/nginx/proy_params

~~~
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
~~~

>

## /etc/nginx/sites-available/yourwebsite.conf

~~~
server {
    listen 443 ssl http2;
    server_name  yourwebsite.com www.yourwebsite.com;
    ssl_certificate /yourssl_certificate/fullchain.pem; # managed by Certbot
    ssl_certificate_key /yourssl_certificate_key/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    error_log /var/log/nginx/app-error.log error;

        location /static {
                alias /var/www/html/yourwebsite/static;
        }

        location / {
                proxy_pass http://localhost:8000;
                include /etc/nginx/proxy_params;
                proxy_redirect off;
                proxy_headers_hash_max_size 784;
                proxy_headers_hash_bucket_size 256;
        }

}
~~~

## /etc/supervisord.conf (at end-of-file)

~~~
[program:aztecglyphrecognition]
directory=/var/www/html/yourwebsite
command=/var/www/html/yourwebsite/aztecglyphvenv/bin/gunicorn --reload -k eventlet -b 0.0.0.0:8000 -w 1 --threads 2 wsgi:app --timeout 1200
user=yourlocaluser
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stderr_logfile=/var/log/app.err.log
stdout_logfile=/var/log/app.out.log
~~~

>

- Supervisor keeps the script running and alive

~~~
sudo supervisorctl start aztecglyphrecognition
~~~

>

- If you don't want to use "supervisor", just activate your virtualenv and run:

~~~
gunicorn --reload -k eventlet -b 0.0.0.0:8000 -w 1 --threads 2 wsgi:app --timeout 1200
~~~

>


# CHANGELOG!

As a continuity of previous [GSoC2021 project](https://summerofcode.withgoogle.com/archive/2021/projects/4769685135949824), there is a list of new implementations and changes:

- Default upload folder changed from app local 'static/uploads/' to '/tmp/aztecglyphrecongitiontempuploads/' (script creates this tmp subfolder if not exists)

## app.py
~~~
#UPLOAD_FOLDER = 'static/uploads/'
UPLOAD_FOLDER = '/tmp/aztecglyphrecognitiontempuploads/'
~~~

>

~~~
#check if upload folder exists and upload images
@app.route('/', methods=['POST'])
def upload_image():
	if os.path.exists(UPLOAD_FOLDER):
        	if len(os.listdir(UPLOAD_FOLDER)) > 0:
                	shutil.rmtree(UPLOAD_FOLDER)
                	os.makedirs(UPLOAD_FOLDER)
	else:
        	os.makedirs(UPLOAD_FOLDER)
~~~

>

- The app gets and shows now the first 6 closed images related with user's upload instead of just 5

## app.py
~~~
def get_closest_images(imga, num_results=6):
        distances = [ distance.cosine(imga, feat) for feat in features ]
        idx_closest = sorted(range(len(distances)), key=lambda k: distances[k])[0:num_results]
        similarity = sorted((int(round((1-(distance.cosine(imga, feat)))*100, 0)) for feat in features), key=int, reverse=True)[0:num_results]
        return idx_closest, similarity 
~~~

>

- Results are printed in screen under 100 px instead 120 for fit with phone devices and some browsers

## /templates/app.html
~~~
img.attr('src', payload['results'][i])
img.attr('width', '100')
img.attr('height', 'auto')
img.attr('style', 'border: 1px solid #ddd')
~~~

>

- Fixed: first most closed result is now readed from array(0) position, instead 1.

## app.py
~~~
idx_closest = sorted(range(len(distances)), key=lambda k: distances[k])[0:num_results]
~~~

>

- Glyph info is obtained from first part of the filename split

## app.py 
~~~
	for idx in results:
		head, tail = os.path.split(images_website[idx])
		file_glyphname = re.split('[0-9]|[_]|[-]|[(]|[,]|[.]', tail, maxsplit=1)
~~~

>

- Each closed image is now also linked on-click to Visual Recognition of Aztec Hieroglyphs site's results

## /templates/app.html
~~~
img.attr('href', 'https://aztecglyphs.uoregon.edu/fulltext-quick-search?search_api_views_fulltext='+payload['glyphname'][i])
~~~

>

- Cosine distance is converted in percentil amount of similarity between user's image and closed image

## app.py
~~~
similarity = sorted((int(round((1-(distance.cosine(imga, feat)))*100, 0)) for feat in features), key=int, reverse=True)[0:num_results]
~~~

>

## /templates/app.html
~~~
Math.abs(payload['similarity'][i])
~~~

>

- A proper match is >85% of similarity (very few times in lower ranks 80-75%)

*Check [Week 10 (15-21 AUG)](https://lisardop.github.io/2022-06-14-weeks-2022/)*

- Glyph name and percentage are shown at image mouse-over

## /templates/app.html
~~~
.attr('title', payload['glyphname'][i]+' ('+ Math.abs(payload['similarity'][i])+'% of similarity)'))
~~~

>

*Check [Week 9 (8-14 AUG)](https://lisardop.github.io/2022-06-14-weeks-2022/) for visual results*

- Cropping glyphs with surrounded color material can give abnomal results due to full-color image comparison

# LICENSE

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
