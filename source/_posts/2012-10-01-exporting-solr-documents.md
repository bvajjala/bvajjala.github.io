---
layout:     post
title:      "Exporting Solr documents"
date:       2012-10-01 12:00:00
categories: [Groovy,Solr]
tags:       [groovy,solr]
published:  true
---

Recently I had to copy some documents from one [Solr][1] server to another. I expected Solr already had an interface that allowed me to extract documents in the same format they were inserted. In that case I would pipe an output of one curl command to another, and consider the job done. As it turned out, the format of Solr input document is different than the output format. Here is how input document looks like:

    <add>
        <doc>
            <field name="id">12345</field>
            <field name="articlestate">published</field>
            <field name="articletype">news</field>
            <field name="body">Lorem ipsum dolor...</field>
            <field name="referenceid">175820</field>
            <field name="referenceid">163786</field>
            <field name="created">2011-02-15T14:57:54.766Z</field>
        </doc>
    </add>

Notice the flat structure of this document: all element names are the same regardless of the filed type, and arrays (referenceid) are not grouped. Now compare it to the output format. Here is what you get when you execute a query against a Solr server:

    <response>
        <lst name="responseHeader">
            <int name="status">0</int>
            <int name="QTime">1</int>
            <lst name="params">
                <str name="q">id:12345</str>
            </lst>
        </lst>
        <result name="response" numFound="1" start="0">
            <doc>
                <str name="id">12345</str>
                <str name="articlestate">published</str>
                <str name="articletype">news</str>
                <str name="body">Lorem ipsum dolor...</str>
                <arr name="referenceid">
                    <str>175820</str>
                    <str>163786</str>
                </arr>
                <date name="created">2011-02-15T14:57:54.766Z</date>
            </doc>
        </result>
    </response>

Even if we ignore the response header, the structure of the response/result/doc is not the same as of input document: the element names reflect the types, the arrays are grouped. If you try to add this document to a Solr server, you will get an error "unexpected XML tag", obviously. I googled for couple hours on how to convert an output document to an input, and, to my surprise, didn't find any solution. Therefore I implemented my own converter in Groovy, which solved the problem. I post it [here][2] in case somebody needs it.

Note: You can also use this script to re-index Solr.

[1]: http://lucene.apache.org/solr/
[2]: http://gist.github.com/3813775
