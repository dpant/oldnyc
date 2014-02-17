# How to do various things for Old NYC.

(Suggest opening this file in TextMate or another editor that understands Markdown.)

## Bring up a demo
Start the appengine app locally via:
cd viewer
dev_appserver.py .

Then visit http://localhost:8080/

You can iterate on JS, HTML & CSS as you usually would.
The images won't be correctly-sized or have metadata in the slideshow. To get this to work, run:

./upload_to_appengine.py --pickle_path nyc/photos.pickle --lat_lons_js_path viewer/nyc-lat-lons.js --image_sizes_path nyc-image-sizes.txt
(takes ~10 minutes)

Be sure to flush memcache on the development instance after you do this! To do
so, visit http://localhost:8000/memcache.
I also had to kill devappserver.py (Control-C) and restart it to see the data.


## Iterate on geocoding
It's easiest to do this by iterating on the records.pickle file, which has one
entry per milstein card, rather than one entry per photo.

./generate-geocodes.py --coders milstein --pickle_path nyc/records.pickle --output_format records.js --geocode > /tmp/records.json

This will print out lots of information about incorrect geocodes and eventually print something like:
25574 milstein
25574 (total)
This is out of 43363 total records in the Milstein collection.

By default, this only uses the local "geocache"--it doesn't fetch any geocodes
from Google Maps. If you want to do that, add --use_network:

./generate-geocodes.py --coders milstein --pickle_path nyc/records.pickle --output_format records.js --geocode --use_network > /tmp/records.json

Geocoding is done based on the "Full Address" column of milstein.csv. You can see this by running:

csvcut -c7,15 nyc/milstein.csv

If you want to determine per-borough geocoding coverage, run
./nyc/coverage-by-borough /tmp/records.json


## Regenerate geocodes for the viewer (nyc-neighborhood-photos.js)
To get new geocodes into the frontend, you need to geocode photos.pickle. Do so
with:

./generate-geocodes.py --coders milstein --pickle_path nyc/photos.pickle --output_format lat-lons.js --geocode > viewer/nyc-lat-lons.js

...

## Generate photos.pickle

photos.pickle is like records.pickle, but it duplicates each record across all its photos.
(There are potentially several photos on the Milstein card for each record.)

cd nyc
./crops-to-json.py crops.txt > /tmp/photos.json
./expand-pickle.py records.pickle /tmp/photos.json photos.pickle

## Generate crops.txt
...


## Generate records.pickle from CSV
The original source for the data is a CSV file that Matt K gave me, milstein.csv.

To convert this to a records.pickle file, run:

cd nyc
./csv-to-record.py

This will read "mistein.csv" and create a "records.pickle" file in the nyc directory.