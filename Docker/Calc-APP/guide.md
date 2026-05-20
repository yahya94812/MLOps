build it's image using `docker build -t calc-app:temp .`
run the image as container using : `docker run -p 8080:8080 calc-app:temp`

test it using http://localhost:8080/calc?a=7&b=8&op=mul in the browser.