---
layout:     post
title:      "Simple web application in Clojure"
date:       2012-11-08 12:00:00
categories: [Clojure,Tutorial]
tags:       [clojure,tutorial]
published:  true
---

This blog entry is a micro-tutorial on how to build a simple web application in Clojure. The reason I call it micro will be clear when I introduce the framework we are going to use. This tutorial will be interesting to programmers relatively new to Clojure, but who have some experience with web frameworks in other languages, for instance Spring MVC. The goal of this tutorial is to help you get started with web development in Clojure. Also I want to share my approach to web development in general and in Clojure in particular. This approach is by no means the best way to develop web applications, but because I like to watch how other people write the software, I thought somebody might be interested to see how I do it.

<!-- more -->

## Problem

So what are we going to build? I don't want to build a simplistic web application for the sake of building the application. On the other hand, I want to constrain myself to small feature set to prevent this tutorial from sinking in too many details. After thinking a while I found a problem which looks pretty simple, but at the same time there is a good chance people (including myself in the first place) will actually use the program I'm going to create. And here is the problem.

I have a bunch of articles and e-books sitting in some directory on my home server. To be able to read those books from any computer in my home network, I run the simple Python web-server, which exposes the content of the directory via HTTP. If you are curious, here is the command I'm using:

    $ cd /path/to/your/ebook/dir
    $ python -m SimpleHTTPServer 3030

And here is how the "library" looks like in the browser

{% img center /images/posts/bookshelf1.png %}

This library application is good enough for me, mainly because it's functional. I can easily find the book by skimming the page or using Find command in a browser. But for the purpose of this tutorial I want to make it slightly better. For example, I can split the file names and display the books in a table view, where I can see clearly what the name of the book is, who the author is, and when it was pablished. I can even add sorting as a bonus feature. Basically, I want something like this

{% img center /images/posts/bookshelf2.png %}

As you can imagine, it shouldn't be hard to do this. The file names are already in the form that is easy to parse. So the question is really how to show the table data in a browser. Simple problem, minimum requirements. Let's see how to solve it in Clojure.

## Tools

Clojure, being a Lisp descendant, is a powerful language. That means you can create your own web framework during a weekend, which many people actually do. But I think it's much better to take existing library, promote it, enhance it, fix the bugs if you like it. In Clojure I found such a framework, it's called [Noir][webnoir]. This framework is very small, so small that their developers call it micro-framework, that's why I'm calling this tutorial micro-tutorial. Probably we shouldn't even call it framework at all, library would probably be a better name. The closest analog to Noir in other languages I know is probably [Webmachine][webmachine] in Erlang, or Spring MVC in Java. I wouldn't compare it to Grails or Rails because those things are huge.

Noir is not only small, it's also simple. You can look at their source code and understand how it works without any problem, provided you have some experience with Clojure. As a result, Noir is a perfect tool for the problem we are going to solve.

Without further ado let's see how Noir works. The easiest way to set up a scaffolding of our future application is by using [Leiningen][leiningen]. Leiningen is a Clojure build tool, very similar to Maven. In Maven you would do mvn archetype:generate, in Leiningen you run

    $ lein new noir bookshelf

where bookshelf is the name of our application. This creates a directory called bookshelf where you can find some Noir template files, plus Leiningen project descriptor

    /.gitignore
    /project.clj
    /README.md
    /resources/public/css/
    /resources/public/css/reset.css
    /resources/public/img/
    /resources/public/js/
    /src/bookshelf/models/
    /src/bookshelf/server.clj
    /src/bookshelf/views/common.clj
    /src/bookshelf/views/welcome.clj
    /test/bookshelf/

You can ignore *.gitignore*, it's already configured properly. Before we make our initial checkin, it's a good practice to edit project.clj and README.md to replace FIXME's. We will edit README file again later when we finish the development to provide more information on how to use the application. Before we move to Noir, let's quickly review project.clj file

{% codeblock project.clj lang:clojure %}
(defproject bookshelf "0.1.0-SNAPSHOT"
  :description "Bookshelf site"
  :dependencies [[org.clojure/clojure "1.4.0"]
                 [noir "1.3.0-beta3"]]
  :main bookshelf.server)
{% endcodeblock %}

*project.clj* is a Clojure version of pom.xml (in fact, you can use pom.xml if you want, Clojure perfectly understands it). First two lines are obvious. Dependency entry specifies which JAR files we need to make our application work. In our case we only need two JARs. Leiningen will check Maven central repository as well as [Clojars][clojars] to download the required JARs with all transitive dependencies. The last line in the project descriptor says which namespace contains the main method. In our case it is `bookshelf.server`. You can find the source of this namespace in */src/bookshelf/server.clj* file. Let's take a look at this file

{% codeblock /src/bookshelf/server.clj lang:clojure %}
(ns bookshelf.server
  (:require [noir.server :as server]))

(server/load-views-ns 'bookshelf.views)

(defn -main [& m]
  (let [mode (keyword (or (first m) :dev))
        port (Integer. (get (System/getenv) "PORT" "8080"))]
    (server/start port {:mode mode
                        :ns 'bookshelf})))
{% endcodeblock %}

`server/load-views-ns` specifies the prefixes of the view namespaces that will be loaded by Noir server. In our case those namespaces start with `bookshelf.views`. By default Leiningen generates two view files: */bookshelf/views/common.clj* and */bookshelf/views/welcome.clj*, but you can create more if your project becomes more complex. Since Noir scans namespaces by prefixes, you can even put your files in the nested directories under */bookshelf/views*, no changes in *server.clj* required.

To start Noir server, run

    $ lein run

This will start Jetty web server bound to localhost at port 8080. If you want to change the default port, say to 3030, run the following command

    $ export PORT=3030; lein run

Besides port, you can specify few other parameters such as :mode, :jetty-options, etc. (you can see all available options in the server [source](https://github.com/noir-clojure/noir/blob/master/src/noir/server.clj)). I'll show below how to specify production mode, for example, when we deploy the final application.

If you started the server with the default port, open your browser at [http://localhost:8080](http://localhost:8080/). You should see the Noir's start page

{% img center /images/posts/bookshelf3.png %}

This page by itself contains all you need to get started with Noir, so you can safely stop reading this article, and just follow the instructions on that page. Those who continue reading this tutorial and wondering where that start page is coming from, please open */src/bookshelf/views/welcome.clj*

{% codeblock /src/bookshelf/views/welcome.clj lang:clojure %}
(ns bookshelf.views.welcome
  (:require [bookshelf.views.common :as common]
            [noir.content.getting-started])
  (:use [noir.core :only [defpage]]))

(defpage "/welcome" []
  (common/layout
    [:p "Welcome to bookshelf"]))
{% endcodeblock %}

Third line tells us that the source of the start page is in *noir/content/getting-started.clj* file. If you are curious where this file is, look [here](https://github.com/noir-clojure/noir/blob/master/src/noir/content/getting_started.clj). Search for `(defpage "/" [] …)` to see how the start page is defined. On your web-site you probably want the start page to be different, so you can remove `[noir.content.getting-started]` from `:require` section.

The next thing to notice on the snippet above is `(defpage "/welcome" [] …)` function. That's how you define URL mappings (or routes, in Noir lingo) of your web-site. (Internally, Noir uses [Compojure][compojure] library to handle the routing.) It is similar to @RequestMapping annotations in Spring-MVC, where you specify which method is called when a user hits the given URL. As you can see, we have only one mapping at the moment, `/welcome`. Since we are building bookshelf application, let's rename it to `/books`. Also, to be even more explicit, let's rename the whole file to *books.clj*. Don't forget to update the namespace. Your *books.clj* file should now look like this

{% codeblock /src/bookshelf/views/books.clj lang:clojure %}
(ns bookshelf.views.books
  (:require [bookshelf.views.common :as common])
  (:use [noir.core :only [defpage]]))

(defpage "/books" []
  (common/layout
    [:p "Welcome to bookshelf"]))
{% endcodeblock %}

If you go to [http://localhost:8080/books](http://localhost:8080/books) in your browser, you should see this screen

{% img center /images/posts/bookshelf4.png %}

By looking at the source of this page, you find it a proper HTML with head and body elements. Those are generated by [Hiccup][hiccup] library, which we'll discuss in a moment. One thing I want to mention about defpage is that you can get the same result if you change `/books` route definition as follows

    (defpage "/books" []
      "<html>
         <head>
           <title>bookshelf</title>
         </head>
         <body>
           <p>Welcome to bookshelf</p>
         </body>
       </html>")

It is just a theoretical exercise, in reality nobody hard-codes the entire HTML inside the Clojure code.

Now let's take look at how the page content is generated. If you look at the routing definition in *books.clj*, you see that the body of `defpage` function is a call to `layout` function defined in */src/bookshelf/views/common.clj*. Let's open this file

{% codeblock /src/bookshelf/views/common.clj lang:clojure %}
(ns bookshelf.views.common
  (:use [noir.core :only [defpartial]]
        [hiccup.page :only [include-css html5]]))

(defpartial layout [& content]
  (html5
    [:head
     [:title "bookshelf"]
     (include-css "/css/reset.css")]
    [:body
     [:div#wrapper
      content]]))
{% endcodeblock %}

`layout` is just a wrapper on top of `hiccup.core/html` function. Hiccup is an XML/HTML rendering library in Clojure. The idea behind it is pretty simple: You build a tree using Clojure vectors, and Hiccup transforms it to a valid HTML. If you are familiar with Groovy MarkupBuilder, it's the same idea. For example, let's define a couple of trees: head and body

    (def head
      [:head
       [:title "bookshelf"]])
    (def body
      [:body
       [:div
        [:p "Welcome to bookshelf"]]])

Here is what you see in REPL when it evaluates different Hiccup HTML formats (I pretty formatted them for visibility purposes)

    (hiccup.page/html5 head body)
    ;=> "<!DOCTYPE html>
    <html>
      <head>
        <title>bookshelf</title>
      </head>
      <body>
        <div><p>Welcome to bookshelf</p></div>
      </body>
    </html>"

    (hiccup.page/html4 head body)
    ;=> "<!DOCTYPE html PUBLIC \"-//W3C//DTD HTML 4.01//EN\" \"http://www.w3.org/TR/html4/strict.dtd\">
    <html>
      <head>
        <title>bookshelf</title>
      </head>
      <body>
        <div><p>Welcome to bookshelf</p></div>
      </body>
    </html>"

    (hiccup.page/xhtml head body)
    ;=> "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Strict//EN\" \"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd\">
    <html xmlns=\"http://www.w3.org/1999/xhtml\">
      <head>
        <title>bookshelf</title>
      </head>
      <body>
        <div><p>Welcome to bookshelf</p></div>
      </body>
    </html>"

Noir's default format is html5, the first example above. You can change it to any other format if needed.

The last piece of *common.clj* I want to mention is `(include-css "/css/reset.css")`. This is another function from Hiccup library. It generates `<link href="/css/reset.css" rel="stylesheet" type="text/css">` element and inserts it in the `head` element of the page. If you recall the scaffolding we generated at the beginning, there is a directory called */resources/public* where Noir keeps CSS files, JavaScript, and images required by your web-site. By default Noir creates *reset.css* in the corresponding subdirectory. Later we'll create other stylesheets and update *common.clj* appropriately.

Now, after we covered all the basics, we are ready to build our application.

## Controller and View

Let's build our view and controller first. While doing that we'll figure out what data we need from the back-end. That's called top-down design.

We define controllers and views in *books.clj* file. For now I also include the model in this file. We'll move it to a proper namespace later when we are done with the front-end. Here is the new version of *books.clj*

{% codeblock /src/bookshelf/views/books.clj lang:clojure %}
(ns bookshelf.views.books
  (:require [bookshelf.views.common :as common])
  (:use [noir.core :only [defpage]]
        [hiccup.element :only [link-to]]))

(defn books []
  [{:author "Fogus M., Houser C."
    :title "The Joy Of Clojure"
    :year "2011"
    :format "pdf"
    :id 1}
   {:author "Fogus M., Houser C."
    :title "The Joy Of Clojure"
    :year "2011"
    :format "epub"
    :id 2}])

(defn- list-books []
  [:table
   [:thead
    [:tr
     [:th "Author"]
     [:th "Title"]
     [:th "Published"]
     [:th "Format"]]]
   (into [:tbody]
         (for [book (books)]
           [:tr
            [:td (:author book)]
            [:td (:title book)]
            [:td (:year book)]
            [:td (link-to (clojure.string/join "/" ["/books" (:id book) (:format book)])
                          (:format book))]]))])

(defpage "/books" []
  (common/layout
    (list-books)))
{% endcodeblock %}

There are three functions in this file: `books` (model), `list-books` (view), and `/books` (controller/router). The controller is essentially the same as before and the model is simply a vector of maps. Each map contains the same keys as column names on the book table. View might look complicated, but there shouldn't be any surprise here either — it's again an ordinary tree. The only new element here is `link-to` function defined in `hiccup.element` namespace. We could really build it directly using `[:a {:href …}]` vector, but link-to is a standard way to do it in Hiccup.

If you refresh [http://localhost:8080/books](http://localhost:8080/books) now (Leiningen/Jetty supports hot redeployment), you will see this screen

{% img center /images/posts/bookshelf5.png %}

The important part of the view function is the format of the links. For the current book model the links are [http://localhost:8080/books/1/pdf](http://localhost:8080/books/1/pdf) and [http://localhost:8080/books/2/epub](http://localhost:8080/books/2/epub). You can verify it by hovering the mouse over them. Those links are actually the main goal of our application. By clicking the link I want to load (or download) the book into the browser and read it. To implement this feature add the following functions to *books.clj*

{% codeblock /src/bookshelf/views/books.clj (cont'd) lang:clojure %}
(defn get-file [id]
  nil)

(defn- ctype [format]
  (if (= "pdf" format) "application/pdf" "text/plain"))

(defpage "/books/:id/:format" {:keys [id format]}
  (content-type (ctype format) (java.io.ByteArrayInputStream. (get-file id))))
{% endcodeblock %}

Ignore for a moment `get-file` function — it does nothing. Later we will move it to model namespace and implement it properly, but we need some placeholder now to compile the application. ctype is a helper function that maps the file format to HTTP content type. I have three ebook formats in my library: ePub, Mobi, and PDF. The first two are plain text formats from HTTP content type perspective. Only PDF requires special type.

The interesting part of the snippet is the controller definition. If you ever implemented REST in Spring MVC, you should see the familiar pattern here. Like Spring, Noir (via Compojure) supports path variables. If you click on [http://localhost:8080/books/1/pdf](http://localhost:8080/books/1/pdf) link in the browser, Noir calls the corresponding controller and binds `id` variable to "1" and `format` variable to "pdf". When we implement `get-file` function, it should return the file with the given id as an array of bytes. Controller then wraps the array into a stream, and Noir pushes it to HTTP response. `content-type` function is defined in *noir.response* namespace, so we need to add `[noir.response :only [content-type]]` to `:use` section at the top of *books.clj*.

That's it, in terms of functionality the controller and the view of our application are done.

## Model

Now we need to extract the logic that creates a model from presentation tier to middle-tier. In Noir that's what models directory is for. In our case it's */src/bookshelf/models*. Let's create a file called *db.clj* in that directory, and move there books and `get-file` functions from *bookshelf.views.books* namespace. We have to update *books.clj* to load the new namespace. It should now look like this

{% codeblock /src/bookshelf/views/books.clj lang:clojure %}
(ns bookshelf.views.books
  (:require [bookshelf.models.db :as db]
            [bookshelf.views.common :as common])
  (:use [noir.core :only [defpage]]
        [noir.response :only [content-type]]
        [hiccup.element :only [link-to]]))

(defn- list-books []
  [:table
   [:thead
    [:tr
     [:th "Author"]
     [:th "Title"]
     [:th "Published"]
     [:th "Format"]]]
   (into [:tbody]
         (for [book (db/books)]
           [:tr
            [:td (:author book)]
            [:td (:title book)]
            [:td (:year book)]
            [:td (link-to (clojure.string/join "/" ["/books" (:id book) (:format book)])
                          (:format book))]]))])

(defpage "/books" []
  (common/layout
    (list-books)))

(defn- ctype [format]
  (if (= "pdf" format) "application/pdf" "text/plain"))

(defpage "/books/:id/:format" {:keys [id format]}
  (content-type (ctype format) (java.io.ByteArrayInputStream. (db/get-file id))))
{% endcodeblock %}

If you refresh your browser, nothing should change.

All right, now it's time to implement our model properly. Recall that our model should scan the directory it's running in for the files with the format Author-Title-Year.FileFormat, and convert each of those files to byte array. Here is how I implemented it

{% codeblock /src/bookshelf/models/db.clj lang:clojure %}
(ns bookshelf.models.db
  (:use [clojure.java.shell :only (sh)]))

(defn- list-files [dir]
  (clojure.string/split (:out (sh "ls" dir)) #"\n"))

(defn- parse [file]
  (when-let [match (re-matches #"([^-]+)-([^-]+)-(\d+)\.(\S+)" file)]
    (zipmap [:file :author :title :year :format] match)))

(defn- add-id [book]
  (assoc book :id (clojure.string/replace (:file book) #"[., ]" "")))

(defn books []
  (->> (list-files ".") (map parse) (remove nil?) (map add-id)))

(defn- file [id]
  (let [mapping (into {} (for [b (books)] [(:id b) (:file b)]))]
    (get mapping id)))

(defn get-file [id]
  (with-open [input  (java.io.FileInputStream. (file id))
              buffer (java.io.ByteArrayOutputStream.)]
    (clojure.java.io/copy input buffer)
    (.toByteArray buffer)))
{% endcodeblock %}

Let's take a look what each of the functions does. Function `list-files` returns a vector of file names that reside in the given directory `dir`. To find all files in the directory I'm using `clojure.java.shell/sh` function which executes `ls` command. This works fine on Linux and Mac, but probalby doesn't on Windows. Function `parse` checks if the given file name has the required format. If so, it returns a map `{:file file, :author Author, :title Title, :year Year, :format FileFormat}`, otherwise it returns `nil`. `add-id` function removes dots, commas, and spaces from the file name and add the result as a book ID to the book map. Function `books` is just a composition of those three functions, and it returns the result expected by the view.

Function `file` returns the file by given ID. The implementation above is not efficient, but my library is too small to notice any performance issues. Finally, `get-file` function finds the file by ID and returns it as a byte array. Those four lines is a pretty standard idiom which you can find in many Clojure source files.

Now we are ready to test our application. For testing purposes I'm going to copy a couple of e-books I recently received updates for to the project home directory. The content of this directory looks like this

    .gitignore
    README.md
    Thomas D.-Programming Ruby 1.9-2010.epub
    Thomas D.-Programming Ruby 1.9-2010.pdf
    project.clj
    resources
    src
    test

I refresh my browser and here I can see these two books

{% img center /images/posts/bookshelf6.png %}

If I click on pdf, I can read the book in my browser

{% img center /images/posts/bookshelf7.png %}

OK, the application is functional. The next step is to make it little bit prettier.

## Styling

Since our application is written in Noir framework, let's make it look like Noir. First, I downloaded Noir background [image](http://www.webnoir.org/img/bg.png) and save it to */resources/public/img* directory. Second, I created a stylesheet */resources/public/css/noir.css* which resembles Noir's original

{% codeblock /resources/public/css/noir.css lang:css %}
body {
    background: #2a2b2b;
    color: #d1d9e1;
    background: url('/img/bg.png');
    padding: 60px 60px;
    font-family: 'Helvetica Neue',Helvetica,Verdana;
}
a {
    text-decoration: none;
    color: #d1d9e1;
}
a:hover {
    color: #6bffbd;
}
h1 {
    color: #6bffbd;
}
{% endcodeblock %}

Then I updated *bookshelf.views.common* namespace to include new CSS

{% codeblock /src/bookshelf/views/common.clj (cont'd) lang:clojure %}
(defpartial layout [& content]
  (html5
    [:head
     [:title "Bookshelf"]
     (include-css "/css/reset.css")
     (include-css "/css/noir.css")]
    [:body
     [:div#wrapper
      content]]))
{% endcodeblock %}

Finally, I want to add a header to the page in *bookshelf.views.books*

{% codeblock /src/bookshelf/views/books.clj (cont'd) lang:clojure %}
(defpage "/books" []
  (common/layout
    [:h1 "Books"]
    (list-books)))
{% endcodeblock %}

Refresh books web page on the browser to see the changes

{% img center /images/posts/bookshelf8.png %}

The last thing left unstyled is the book table. I won't style it directly, because I want to add client-side sorting to it, and I happen to know that TableSorter JavaScript library provides its own style.

## JavaScript

TableSorter is a jQuery plugin, so you need to download jQuery first. Grab the latest [min](http://code.jquery.com/jquery-1.8.2.min.js) and save it to */resources/public/js* directory. Then, download [tablesorter.zip](http://tablesorter.com/docs/#Download) that contains both JavaScript and stylesheet files. As before, JavaScript goes to */resources/public/js* and stylesheets go to */resources/public/css* directory. Here is the resources directory structure I have after everything is saved

    resources/public/css/tablesorter/asc.gif
    resources/public/css/tablesorter/bg.gif
    resources/public/css/tablesorter/desc.gif
    resources/public/css/tablesorter/style.css
    resources/public/js/jquery-1.8.2.min.js
    resources/public/js/jquery.tablesorter.js
    
If you are curious, here is my *style.css*. I changed the original tablesorter css a little bit to better fit Noir theme

{% codeblock /resources/public/css/style.css lang:css %}
table.tablesorter {
    margin:10px 0pt 15px;
    font-size: 10pt;
    width: 100%;
    text-align: left;
}
table.tablesorter thead tr th, table.tablesorter tfoot tr th {
    color: #000000;
    background-color: #b0b8c0;
    border: 1px solid #2a2b2b;
    font-size: 10pt;
    padding: 4px;
}
table.tablesorter thead tr .header {
    background-image: url(bg.gif);
    background-repeat: no-repeat;
    background-position: center right;
    cursor: pointer;
}
table.tablesorter tbody td {
    color: #b0b8c0;
    padding: 4px;
    background-image: url('/img/bg.png');
    vertical-align: top;
}
table.tablesorter tbody tr.odd td {
    background-color:#F0F0F6;
}
table.tablesorter thead tr .headerSortUp {
    background-image: url(asc.gif);
}
table.tablesorter thead tr .headerSortDown {
    background-image: url(desc.gif);
}
table.tablesorter thead tr .headerSortDown, table.tablesorter thead tr .headerSortUp {
    background-color: #6bffbd;
}
{% endcodeblock %}

To enable tablesorter we have to update both view files. Few changes in bookshelf.views.common

{% codeblock /src/bookshelf/views/common.clj lang:clojure %}
(ns bookshelf.views.common
  (:use [noir.core :only [defpartial]]
        [hiccup.page :only [include-css include-js html5]]))

(defpartial layout [& content]
  (html5
    [:head
     [:title "Bookshelf"]
     (include-js "/js/jquery-1.8.2.min.js")
     (include-js "/js/jquery.tablesorter.js")
     (include-css "/css/reset.css")
     (include-css "/css/tablesorter/style.css")
     (include-css "/css/noir.css")]
    [:body
     [:div#wrapper
      content]]))
{% endcodeblock %}

and few changes in *bookshelf.views.books*

{% codeblock /src/bookshelf/views/books.clj lang:clojure %}
(ns bookshelf.views.books
  (:require [bookshelf.models.db :as db]
            [bookshelf.views.common :as common])
  (:use [noir.core :only [defpage]]
        [noir.response :only [content-type]]
        [hiccup.element :only [link-to javascript-tag]]))

(defn- list-books []
  [:table.tablesorter {:id "bookTable"}
   [:thead
    [:tr
     [:th "Author"]
     [:th "Title"]
     [:th "Published"]
     [:th "Format"]]]
   (into [:tbody]
         (for [book (db/books)]
           [:tr
            [:td (:author book)]
            [:td (:title book)]
            [:td (:year book)]
            [:td (link-to (clojure.string/join "/" ["/books" (:id book) (:format book)])
                          (:format book))]]))])

(defpage "/books" []
  (common/layout
    [:h1 "Books"]
    (javascript-tag "$(document).ready(function() {$(\"#bookTable\").tablesorter();});")
    (list-books)))

(defn- ctype [format]
  (if (= "pdf" format) "application/pdf" "text/plain"))

(defpage "/books/:id/:format" {:keys [id format]}
  (content-type (ctype format) (java.io.ByteArrayInputStream. (db/get-file id))))
{% endcodeblock %}

After we refresh the browser, we should see the final design and be able to sort the table

{% img center /images/posts/bookshelf9.png %}

## Ship it!

OK, we are ready to ship. But before we build a deployable artifact, we, as professional developers, should update documentation ([README.md](https://github.com/ndpar/bookshelf/blob/master/README.md) in our case) and finalize the version of the application

{% codeblock project.clj lang:clojure %}
(defproject bookshelf "0.1.0"
  :description "Bookshelf site"
  :dependencies [[org.clojure/clojure "1.4.0"]
                 [noir "1.3.0-beta3"]]
  :main bookshelf.server)
{% endcodeblock %}

I want to run this application as a a standalone Java web application, without any dependency on Leiningen. Therefore I have to add `:gen-class` to *server.clj*

{% codeblock /src/bookshelf/server.clj lang:clojure %}
(ns bookshelf.server
  (:require [noir.server :as server])
  (:gen-class))

(server/load-views-ns 'bookshelf.views)

(defn -main [& m]
  (let [mode (keyword (or (first m) :dev))
        port (Integer. (get (System/getenv) "PORT" "8080"))]
    (server/start port {:mode mode
                        :ns 'bookshelf})))
{% endcodeblock %}

Now package the application by running the following command

    $ lein uberjar

As a result of this command, *bookshelf-0.1.0-standalone.jar* artifact is created in the target directory. I scp this file to my server, to the directory where my books are located, and start the app

    $ export PORT=3030; nohup java -jar bookshelf-0.1.0-standalone.jar prod > nohup.out 2>&1 &

And that's basically it. We have created a simplest web application in Clojure, which might be useful by its own. But more importantly, you've learned *how* to build it. I hope you enjoyed reading this tutorial as I enjoyed writing it.

## Recap

If you want to create web application in Clojure, try Noir. Noir is small and easy to pick up.

- use defpage macro to define URL routes
- use defpartial macro to build views
- use Leiningen to run local web server
- use REPL to experiment with business logic
- have fun with Clojure

## Resources

1. Long in-depth Noir tutorial: [http://yogthos.net/blog/22](http://yogthos.net/blog/22)
1. Source code of this tutorial: [https://github.com/ndpar/bookshelf](https://github.com/ndpar/bookshelf)
1. Noir: [http://www.webnoir.org][webnoir]
1. Hiccup: [https://github.com/weavejester/hiccup][hiccup]

P.S. After I wrote the draft of this post I found another [tutorial](http://clojure-lab.tumblr.com/post/35207280637/clojure-web-development) on Clojure web development, which even has the same name for the application! If I knew about it before, I wouldn't probably write mine. But since it's already typed, let it be published.


[webnoir]: http://www.webnoir.org/
[webmachine]: http://wiki.basho.com/Webmachine.html
[leiningen]: http://leiningen.org/
[clojars]: https://clojars.org/
[compojure]: https://github.com/weavejester/compojure
[hiccup]: https://github.com/weavejester/hiccup
