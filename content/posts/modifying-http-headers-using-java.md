---
title: "Modifying Http Headers Using Java"
date: 2010-06-12 18:00:13.000000000 -04:00
draft: false
tags:
    - Coding
    - DHTTP Headers
    - Java
    - Programming
keywords:
    - Technology
---
There can be times when you are asked to do the impossible, thats  what we do as developers, we are super-humans and our kryptonite is  caffeine.      
Recently, I came across a project in which I was supposed to modify  the _Http headers_ of the request going to a different server. Like a superhuman, I  thought it was a piece of cake; But after a lot of brainstorming,  brute-forcing and caffeine intake I realized it wasn’t that simple.  In-fact it was not possible at all. Now, before you folks get mad and  swear at me,  let me explain you the problem and how I tried to solve  it, how miserably I failed and  finally how I got around it by a simple  trick.      

### Problem:
Let us start with the problem. What I had to do was simple, upon a user  event (assume a mouse click) I needed to calculate a value and insert it  into the header of a new request. Note the ‘new request’ here. This new  request is not on the same server but on a different server where I  have very little control.      

### Conclusion:
I spent considerable amount of time working on this problem, if only I  knew the simple answer to the question, it would have saved me so much  of time. The answer was simply you cannot do it ! you cannot change the _headers_ of the request which is redirected to a different server.      

**But why ?**    
The reason is that if you were able to modify the _headers_ you could  intercept the user requests in what ever way you want. There would be a  high probability of man-in-the-middle attack. One of the many ingenious  way of Java to protect you.      

### Workaround:
The work around was to encrypt the information in the url and send it,  or use a form or hidden field.      
Using _HttpServletRequestWrapper_ class:
If you want to modify the _headers_ of the request which is on the same  server then it can be done by overriding the  getHeaderName and  getHeader of the HttpServletRequestWrapper class. Now, if you are  curious like me you might ask why on the earth do I want to use a  wrapper class ? Why cannot I simply use a filter to modify the _headers_.  As it appears, when you redirect or for that matter forward a request  your _headers_are committed and the stream is closed  by using the  _HttpServletRequestWrapper_ class you can tell the servlet to stand by  before committing the response so that we can modify the _headers_.  Enough of theory lets gets some coding done.      

In the rest of the blog I will explain how to modify the _headers_ by overriding  the _HttpServletRequestWrapper_ class.
![ModifyHttpHeaders!](/static/images/ModifyHttpHeaders-1.png)

**Assumptions**     
Here, I am assuming you are familiar with the basics of J2EE filter and  have already set up one for testing this example. [Filter  example](https://www.tutorialspoint.com/jsp/jsp_writing_filters.htm) is a good filter example to start with.      
Once you have set up a filter, lets call it MyFilterExample, it  should look pretty much like the following code.

```
package com.sandeepmore.filters;

import java.io.IOException;
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpServletResponseWrapper;
import javax.servlet.http.HttpSession;

import java.util.logging.*;

public class MyFilterExample implements javax.servlet.Filter {
private ServletContext servletContext;
private Logger log;

public MyFilterExample(){
super();
}

public void init(FilterConfig filterConfig) throws ServletException {
servletContext = filterConfig.getServletContext();
log = Logger.getLogger(MyFilter.class.getName());
}

public void doFilter( ServletRequest req,
ServletResponse res,
FilterChain filterChain)
throws IOException, ServletException {

MyServletRequestWrapper httpReq = new MyServletRequestWrapper((HttpServletRequest)req);
HttpServletResponse httpRes = (HttpServletResponse)res;

HttpSession session = httpReq.getSession();

httpReq.addHeader("myHeader", "value");

filterChain.doFilter(httpReq, httpRes);

}

public void destroy(){
}
}
```


As you can see we have added our custom _headers_ using the line     
`httpReq.addHeader("myheader", "value");`     
The entire meat is in the method doFilter(). In the above class this  is the only method which needs to be modified as shown above. All we did  in this method is wrapped the _http_ request object with the  _MyServletRequestWrapper_ class (which we are about to see in a while) so  that we can override the getHeaderName and getHeader methods and insert  our custom __headers_headers_. Code for MyServletRequestWrapper class is as follows       

```
package myfilter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.util.*;

public class MyServletRequestWrapper extends HttpServletRequestWrapper{

private Map headerMap;

public void addHeader(String name, String value){
headerMap.put(name, new String(value));
}

public MyServletRequestWrapper(HttpServletRequest request){
super(request);
headerMap = new HashMap();
}

public Enumeration getHeaderNames(){
HttpServletRequest request = (HttpServletRequest)getRequest();
List list = new ArrayList();
for( Enumeration e = request.getHeaderNames() ;  e.hasMoreElements() ; )
list.add(e.nextElement().toString());
for( Iterator i = headerMap.keySet().iterator() ; i.hasNext() ; ){
list.add(i.next());
}
return Collections.enumeration(list);
}
public String getHeader(String name){
Object value;
if((value = headerMap.get(""+name)) != null)
return value.toString();
else
return ((HttpServletRequest)getRequest()).getHeader(name);
}
}
```

All this class does is that it adds the new header and the value  associated with it to a Hashmap, ‘headerMap’ in our case. When the  getHeaderNames method is called the new _headers_ are inserted into a  ArrayList and returned as a Enumeration.     
The getHeader method searches for the header value in Hashmap  ‘headerMap’ first then it checks the HttpServletRequest object for the  header information.     
This is a very simple trick and can be very handy. Again, one  important thing to remember this method is useful only if you are  forwarding a request on the same server and not ‘redirecting’ to a  different server.     
Happy Coding     
/srm     

