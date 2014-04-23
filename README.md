lab8 in go
==========
Based on jrbalsano's lab 8 in Node.js: https://github.com/jrbalsano/lab8

Part 1: Install go
------------------

Installing go on your own machine isn't too difficult. The directions are here:
http://golang.org/doc/install Note that go is an  opinionated language, it wants
things laid out exactly how it wants them. So either do the system install, or
install it in `~/go`. 

If you're using OS X or Linux, it's best to just install the packages they
provide.

If you want to install it on CLIC, we can install it into your home directory
using the tarball method.

```sh
$ cd
$ wget https://go.googlecode.com/files/go1.2.1.linux-amd64.tar.gz
$ tar zxvf go1.2.1.linux-amd64.tar.gz
```

Now one last thing. We need to configure our GOROOT and PATH to include our
install of go. GOROOT is the place where go expects to find itself, and the
programs we write. There is a default system location for this, but since we're
installing it in our home directory we need to specify where to find it, in this
case `~/go/`.

The PATH is a list of directories that your shell searches whenever you type a
program you want to execute. The current directory is never in your path;
that's why when you type things like `mdb-lookup-server` the shell never finds
your program, but when you specify the folder its in using `./mdb-lookup-server`
the shell knows its not going to find it in the directories in PATH, but in the
current directory. Anyway, to add our go install to the GOROOT and PATH, add the
following line your `~/.bashrc` file (or whatever profile you're using - if you
don't know what this means, just edit `~/.bashrc`)

```
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
```

Once you've done this, try the following sequence of commands:

```
source ~/.bashrc
which go
```

If the output says `/home/UNI/go/bin/go` or something equivalent then you're
good to go. If not, try logging out and back in again and then running `which
go`. If all this is good, then congratulations, you've installed go!


One thing that might be nice is syntax highlighting. Because go is a young
language we'll have to add it to vim ourselves. Luckily inside `~/go/misc/vim`
is everything we need (it also has emacs support, but I'm a rational person so I
can't help you with emacs). Read the readme.txt in that directory for how to
install it.


Part 1.5 Test Go
----------------
Let's first get setup with how go likes projects to be organized, and write our
first go file!

In go, everything should be located in subfolders of GOROOT/src. Let's do that
now, and create go's "hello world".

```sh
$ mkdir -p ~/go/src/lab8
$ cd ~/go/src/lab8
$ git init .
$ echo -e "package main\nimport \"fmt\"\n\nfunc main() {\n\tfmt.Printf(\"hello, world\\\n\")\n}\n" > hello.go
$ go run hello.go
```

You should see the go program print out "hello, world"! 

If you want to learn more go, I suggest going through "A Tour of Go" at
http://tour.golang.org . It starts slow, but quickly you'll see some of the cool
things you can do.


Part 2: Serve static files
--------------------------

For part 2 we want to mirror part 2a of lab 7: serve static files from some
directory on some port. I think for fun you should try to exactly match the
interface of lab7's server, so have it take the same parameters as arguments,
etc. Then your go server is a drop in replacement. We have a few choices in
terms of how much we want go's libraries to handle for us, and how much we want
to write ourselves.

A minimal example would use the http module, and its FileServer handler (see the
example in the go docs). This is absurdly simple, in a simple case. We should
put all this in a file, right? Let's call it http-server.go.

```sh
$ cd ~/go/src/lab8/
$ vim http-server.go
```

Now to actually build it, I recommend using `go install`, which creates the
executable and puts it in ~/go/bin/.


```sh
$ go install
$ ~/go/bin/http-server 8080 ~/html
```

You can then test it the same way you tested lab 7, open a browser to
http://CITY.clic.cs.columbia.edu:8080/cs3157/tng/

Once you get it running, include the following in your README.txt:

* The (part of) `ps ajxfww` output that shows your go server running
* The capture of a netcat session fetching the Star Trek index.html page from
  your new go http-server


Part 3: Build a simple app using the http package
-----------------------------------------------

Go includes a fairly robust http package in the standard library, which makes
writing web application servers quite easy! You define a function that implements
http.Handler, and it pretty automatically makes it work. 

Let's take our basic static only server, and add a special path, "/hello"! 

We need to create an HTTP handler, a function that takes
`http.ResponseWriter, r *http.Request` as its arguments. It writes its output to
w. We can base ours on [this example about wiki
pages](http://golang.org/doc/articles/wiki/#tmp_4), but instead of doing any of
the fancy stuff in viewHandler, just have it `Fprintf(w, "Hello, world!")`.

Then we need to register our helloHandler using `http.HandleFunc`. But we still
want it to server static files if we don't request hello - how can we do that?
We can use `http.Handle` as a fallback, which will use http.FileServer for `/`.
Then in the ListenAndServe call we can just use nil.

How can we take this further?
1. Display a hit count. There are a few ways to do this (say, a global variable),
but the neatest is probably using a closure. Take a look at [Kyle's post](https://groups.google.com/forum/#!topic/golang-nuts/SGn1gd290zI), but use an integer instead of the string hi.

2. Display (clearly labeled) the url for the request and the parsed version of
   the url. This information is available in the [Request](http://golang.org/pkg/net/http/#Request) struct. To get the form arguments, you'll need to call ParseForm().

Send the following query from your bowser to see what values are output by your
server: /hello?key=abc

Include the output of the above request in your README.txt along with an
explanation/description of what each field in the parsed uri means. 

Don't forget to git commit as you work! Especially with a lab like this, where
we're building incrementally.



Note, for advanced mode you can use [raw sockets](http://golang.org/pkg/net/)
just like you did in lab 7. It's much friendlier in go, with Dial and Listen.
(ie, listening on a port is a single line of code, instead of the dozens that C
takes). You'll find that doing HTTP with go is basically exactly the same, just
takes less code, and you worry less about strings.


Part 3: mdb-lookup in go
--------------------------

For this part of the lab you're going to implement the same functionality as in
lab7. That is, make your go application respond to /mdb-lookup and 
/mdb-loop?key=search_string URLs in the same way lab 7 did, and also let it 
server files from your public html directory.

Ideally your server is already serving both static files, and hello world on `/hello`. If not, get that working.

Next, let's connect to the mdb-lookup-server. We can look at the overview of the
[net package](http://golang.org/pkg/net/#pkg-overview), and use the Dial
function. It's incredibly easy, just dial a tcp connection to the server and
port that was given as a command line argument. Then you can read and write
to/from the connection, just like a file.

At this point you should be all set - the logic is entirely the same as in C,
but the string manipulation much easier!

Sources
#######
* http://golang.org/
* http://tour.golang.org/
* http://golang.org/doc/articles/wiki/#tmp_3
* http://golang.org/pkg/net/#pkg-overview

