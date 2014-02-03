---
layout:     post
title:      "Parsing files using Groovy regex"
date:       2009-10-22 12:00:00
categories: [Groovy,Regex]
tags:       [groovy,regex]
published:  true
---

In my previous [post][1] I mentioned several ways of defining regular expressions in Groovy. Here I want to show how we can use Groovy regex to find the data in the files.

## Parsing properties file (simplified)<sup>1</sup>

*Data:* each line in the file has the same structure; the entire line can be matched by single regex.

*Task:* transform each line to the object.

*Solution:* construct regex with capturing parentheses, apply it to each line, extract captured data.

*Demonstrates:* `File.eachLine` method, matrix syntax of *Matcher* object.

{% codeblock lang:groovy %}
def properties = [:]
new File('path/to/some.properties').eachLine { line ->
    if ((matcher = line =~ /^([^#=].*?)=(.+)$/)) {
        properties[matcher[0][1]] = matcher[0][2]
    }
}
println properties
{% endcodeblock %}


## Parsing CSV files (simplified)<sup>2</sup>

*Data:* each line in the file has the same structure; the line consists of the blocks separated by some character sequence.

*Task:* transform each line to the list of objects.

*Solution:* construct regex with capturing parentheses, parse each line with the regex in a loop extracting captured data.

*Demonstrates:* `~//` *Pattern* defenition, `Matcher.group` method, `\G` regex meta-sequence.

{% codeblock lang:groovy %}
def regex = ~/\G(?:^|,)(?:"([^"]*+)"|([^",]*+))/
new File('path/to/file.csv').eachLine { line ->
    def fields = []
    def matcher = regex.matcher(line)
    while (matcher.find()) {
        fields << (matcher.group(1) ?: matcher.group(2))
    }
    println fields
}
{% endcodeblock %}


## Finding snapshot dependencies in the POM (simplified)<sup>3</sup>

*Data:* file contains blocks with known boundaries (possibly spanning multiple lines).

*Task:* extract the blocks satisfying some criteria.

*Solution:* read the entire file into the string, construct regex with capturing parentheses, apply the regex to the string in a loop.

*Demonstrates:* `File.text` property, list syntaxt of *Matcher* object, named capture, global `\x` regex modifier, local `\s` regex modifier.

{% codeblock lang:groovy %}
def pom = new File('path/to/pom.xml').text
def matcher = pom =~ $/(?x)
    <dependency>                          \s*
      <groupId>([^<]+)</groupId>          \s*
      <artifactId>([^<]+)</artifactId>    \s*
      <version>(.+?-SNAPSHOT)</version>   (?s:.*?)
    </dependency>
/$
matcher.each { matched, groupId, artifactId, version ->
    println "$groupId:$artifactId:$version"
}
{% endcodeblock %}


## Finding stacktraces in the log

*Data:* file contains entries each of which starts with the same pattern and can span multiple lines. Typical example is log4j log files:

    2009-10-16 15:32:12,157 DEBUG [com.ndpar.web.RequestProcessor] Loading user
    2009-10-16 15:32:13,258 ERROR [com.ndpar.web.UserController] id to load is required for loading
    java.lang.IllegalArgumentException: id to load is required for loading
         at org.hibernate.event.LoadEvent.(LoadEvent.java:74)
         at org.hibernate.event.LoadEvent.(LoadEvent.java:56)
         at org.hibernate.impl.SessionImpl.get(SessionImpl.java:839)
         at org.hibernate.impl.SessionImpl.get(SessionImpl.java:835)
         at org.springframework.orm.hibernate3.HibernateTemplate$1.doInHibernate(HibernateTemplate.java:531)
         at org.springframework.orm.hibernate3.HibernateTemplate.doExecute(HibernateTemplate.java:419)
         at org.springframework.orm.hibernate3.HibernateTemplate.executeWithNativeSession(HibernateTemplate.java:374)
         at org.springframework.orm.hibernate3.HibernateTemplate.get(HibernateTemplate.java:525)
         at org.springframework.orm.hibernate3.HibernateTemplate.get(HibernateTemplate.java:519)
         at com.ndpar.dao.UserManager.getUser(UserManager.java:90)
         ... 62 more
    2009-10-16 15:32:14,659 DEBUG [com.ndpar.jms.MessageListener] Received message:
         ... multi-line message ...
    2009-10-16 15:32:15,169 INFO  [com.ndpar.dao.UserManager] User: ...

*Task:* find entries satisfying some criteria.

*Solution:* read the entire file into the string<sup>4</sup>, construct regex with capturing parentheses and lookahead, split the string into entries, loop through the result and apply criteria to each entry.

*Demonstrates:* regex interpolation, combined global regex modifiers `\s` and `\m`.

{% codeblock lang:groovy %}
def log = new File('path/to/your.log').text
def logLineStart = /^\d{4}-\d{2}-\d{2}/
def splitter = log =~ $/(?xms)
    (    ${logLineStart}   .*?)
    (?=  ${logLineStart} | \Z)
/$
splitter.each { matched, entry ->
    if (entry =~ /(?m)^(?:\t| {8})at/) println entry
}
{% endcodeblock %}


### Resources

- Groovy [regexes][2]
- Groovy [one-liners][3]
- Using [String.replaceAll][4] method


### Footnotes

1. This example is for demonstration purposes only. In real program you would just use `Properties.load` method.
2. The regex is simplified. If you want the real one, take a look at Jeffrey Friedl's [example][5].
3. Again, in reality you would find snapshots using `mvn dependency:resolve | grep SNAPSHOT` command.
4. This approach won't work for big files. Take a look at this [script][6] for practical solution.


[1]: /2009/09/25/groovy-regular-expressions/
[2]: http://groovy.codehaus.org/Regular+Expressions
[3]: http://groovy.codehaus.org/Groovy+CLI
[4]: http://mrhaki.blogspot.com/2009/10/groovy-goodness-using-replaceall.html
[5]: http://regex.info/listing.cgi?ed=3&p=401
[6]: http://github.com/ndpar/utilities/blob/master/stacktraces.groovy
