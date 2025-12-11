# nginx files for worker nodes
Use the commands below to create the nginx files

## worker 1

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <meta http-equiv="x-ua-compatible" content="ie=edge">
        <meta name="referrer" content="always">
        <meta name="description" content="Stories and photos by Phil Smrek">
        <title>K8s nginx</title>
        <style>
            body {
                background-color: darkseagreen;
            }
        </style>
    </head>
    <body>
        <h1>Hello from k8s-w1</h1>
        <p>This is worker node 1</p>
    </body>
</html>
```

## worker 2

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <meta http-equiv="x-ua-compatible" content="ie=edge">
        <meta name="referrer" content="always">
        <meta name="description" content="Stories and photos by Phil Smrek">
        <title>K8s nginx</title>
        <style>
            body {
                background-color: lightskyblue;
            }
        </style>
    </head>
    <body>
        <h1>Hello from k8s-w2</h1>
        <p>This is worker node 2</p>
    </body>
</html>
```
