---
layout: page
title: Blog
---

<p>
	<h5> Technical blogs on security around Big Data using <a href="http://knox.apache.org/">Apache Knox</a> </h5>
</p>
<hr/>
<ul>
    <li>
      <a href="https://cwiki.apache.org/confluence/display/KNOX/2017/03/01/Apache+Knox+using+multiple+LDAP+Realms">Apache Knox using multiple LDAP Realms</a>
	  <p class="date">Mar 01, 2017</p>
    </li>
    <li>
      <a href="https://cwiki.apache.org/confluence/display/KNOX/2017/02/24/Hadoop+Auth+%28SPNEGO+and+delegation+token+based+authentication%29+with+Apache+Knox">Hadoop Auth (SPNEGO and delegation token based authentication) with Apache Knox</a>
	  <p class="date">Feb 24, 2017</p>
    </li>
    <li>
      <a href="https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=66854729">Enhanced LDAP Authentication using PAM+SSSD+LDAP in Apache Knox</a>
	  <p class="date">Dec 02, 2016</p>
    </li>
</ul>

<p>
	<h4>Blog about technical and not technical stuff</h4>
</p>
<hr/>
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
	  <p class="date">{{ post.date | date: "%b %d, %Y" }}</p>
    </li>
  {% endfor %}
</ul>