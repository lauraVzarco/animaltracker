# Animal Tracker

> A tool for manual tracking and tagging of animals in camera trap images

## Usage example

1. Open a set of images by clicking ```Browse..``` and selecting multiple image files.
1. Use the left and right arrow keys (or click the previous and next buttons) to move through the sequence.
1. Start a new track by clicking the ```Add sequence``` button and clicking on a point in the image.
1. To track the same point in the sequence, move to a different frame and click again. Points can only be added whilst in 'Add mode' and only one point can be added per sequence on a single frame. To move any already added points then click on the ```Add mode``` button to switch into 'Edit mode' and click and drag the points to move them. 
1. Data can be exported to a CSV file by using the button at the top.

Information on all tracks added can be seen underneath the image window. 

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).
