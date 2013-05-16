---
layout: post
title: Key Concepts in Person-Based Analytics
categories: getting-started
tags: [identities]
author: Eric Fung
summary: Key concepts about person-based analytics you should know before using KISSmetrics.
---

<hr />
# Table of Contents

* [Section 1. People Are Represented by Two Types of "Identities"][section1]
* [Section 2. Use the Named ID You Have Access To][section2]
	* [Section 2.1 Setting Named ID's: `identify` method][section2.1]
	* [Section 2.2 Example Uses of `identify` to Avoid][section2.2]
* [Section 3. An ID Should Not Change Often, But When It Does...][section3]
 	* [Section 3.1 Linking Two IDs: `alias` method][section3.1]
	* [Section 3.2 Let Our JS Library Link Two IDs][section3.2]
	* [Section 3.3 JS Library Examples][section3.3]
	* [Section 3.4 Word of Warning][section3.4]


In KISSmetrics, every unit of recorded data is associated with the person who interacted with your website or application. This is a seemingly simple idea but it has powerful consequences.

<a name="section-1"></a>
## 1. People Are Represented by Two Types of "Identities"

Your visitors and customers can be split into two groups: those you have information about, and those you don't. These people can be represented with two types of "identities". Some terminology:

* ***Named Identities***: a *recognizable* string or int that represents a person:
  * `foo@example.com`
  * `User #12345`
  * `Facebook ID #67890`
* ***Anonymous Identities***: a *randomly-generated* string or int that represents a person before you know who they are:
  * `y75Fe33597qBqkR4obZZYV+wF3Y=`
  * `6j1KH1zrwBS6T2iIsixvpfnCnxY=`

You can segment nearly every report in KISSmetrics by the "**Customer ID**" property. This will tell you the per-person breakdown of the results you are seeing.

<a name="section-2"></a>
## 2. Use the Named ID You Have Access To

Ideally, each customer needs to be represented by only **one** Named ID. Whatever type of ID you use depends on the information your app/website collects. The most common Customer IDs are:

* **Account ID**: useful if people sign up for your service and log in.
* **Email Address**: useful if people subscribe to mailing lists.

You will have the chance to add "secondary" identities as additional KISSmetrics properties. So if your team wants to segment reports by both Account ID *and* Email Address, you can maintain Account IDs as the Named Identity (or Customer ID), while setting Email Address as a separate KISSmetrics *Property* of each person.

Notice that *First Name/Last Name* and *IP Addressses* are not considered strong enough identifiers to distinguish one person from another. However, you can still save this type of metadata as KISSmetrics Properties.

Setting Properties will be discussed in the [API Methods area][common-methods].

<a name="section-2.1"></a>
### 2.1 Setting Named ID's: `identify` method

When you use our `identify` method, you indicate who is doing the next set of events, until you call `identify` again.

When you call `identify` with our JavaScript Library, the ID is saved to a first-party cookie that is used for this and future browsing sessions. The cookie expires after 5 years unless the visitor clears their cookies.

Example uses of `identify`:
{% highlight js+ruby %}
// JS + Ruby, assuming your @account object has a 'username' field.
_kmq.push(['identify', '<%= @account.username ? @account.username : nil %>']);
{% endhighlight %}

{% highlight js+php %}
// Wordpress
_kmq.push(['identify', '<?php $current_user = wp_get_current_user(); if ( ($current_user instanceof WP_User) ) echo $current_user->user_login; ?>']);
{% endhighlight %}

<a name="section-2.2"></a>
### 2.2 Example Uses of `identify` to Avoid

Calling `identify` in these ways is counterproductive:
{% highlight js %}
// This is a JS Syntax error. The argument should be a string or an int.
_kmq.push(['identify', steve@apple.com ]);

// These treat ALL your visitors as one person, someone named "anonymous" or "guest"
_kmq.push(['identify', 'anonymous' ]);
_kmq.push(['identify', 'guest' ]);
{% endhighlight %}

<a name="section-3"></a>
## 3. An ID Should Not Change Often, But When It Does…

Your customers can accumulate several identities over their lifetime of interaction with your application or website. They will initially visit anonymously, possibly from multiple web browsers. On each of those browsers, they will be first represented using Anonymous ID's. Once they sign up or log in or otherwise "Convert", you can continue to track their activity under their Named ID, which is much more meaningful.

Thus, a person should have no more than one Named ID, but can have several Anonymous IDs. Regardless of how many IDs a person accumulates, let's make sure they all refer back to the same KM Person entity.

<a name="section-3.1"></a>
### 3.1 Linking Two IDs: `alias` method

KISSmetrics Aliasing connects 2 IDs. Following an alias, data from both IDs represents the same person.

Example uses of `alias`:

{% highlight js+ruby %}
// Calling alias with explicit arguments
_kmq.push(['alias', 'y75Fe33597qBqkR4obZZYV+wF3Y=', 'foo@example.com' ]);

// Dynamically filling in the arguments
_kmq.push(['alias', KM.i(), '<%= @account.username ? @account.username : nil %>' ]);
{% endhighlight %}

(`KM.i()` is a function of our JS Library that [returns the user's current KM ID][js-ids].)

<a name="section-3.2"></a>
### 3.2 Let Our JS Library Link Two IDs

Because aliasing is so core to KISSmetrics, our JavaScript Library attempts to automate aliasing when it's appropriate so that you rarely have to explicitly use that method.

* Our JavaScript Library will generate Anonymous Identities for people it has not seen before. (The generated ID is Base64 encoded, so the IDs will contain the characters `a-z, A-Z, 0-9, +, /, and =`.)

If the visitor is currently **anonymous**, these 3 interactions will simultaneously establish the visitor's Named ID going forward, as well as *automatically alias the Named ID to their previous Anonymous ID*:

1. You call `identify` with an argument, like in the examples above.
2. The visitor submits a form you are tracking (`trackSubmit`), and the form has an `<input>` that can be used as an identifier. [See here for more details][id-from-form].
3. The visitor views a URL tagged with the `kmi` query parameter. This is our [URL API][url-api].

<a name="section-3.3"></a>
#### 3.3 JS Library Examples

Suppose a visitor arrives and is given the Anonymous ID `1rNfguj/Kk3VsDyuLBXqiOnzYK4=`.

In each of these scenarios, our JS Library would remember that this person is `foo@example.com` for this session and future sessions, ***and*** alias the two IDs together:

`1rNfguj/Kk3VsDyuLBXqiOnzYK4=` --> `foo@example.com`

* The visitor's browser executes this JS:

{% highlight js %}
_kmq.push(['identify', 'foo@example.com' ]);
{% endhighlight %}

* The visitor submits this form, which is being tracked:

{% highlight html %}
<form id="login_form" onsubmit="login()">
  <input name="email" type="text"></input>
  <input name="password" type="password"></input>
</form>
<script type="text/javascript">
  _kmq.push(['trackSubmit', 'login_form', 'Logged In']);
</script>
{% endhighlight %}

* Visiting the URL `http://www.example.com/?kmi=foo%40example.com`

<a name="section-3.4"></a>
### 3.4 Word of Warning

Aliasing is powerful, but can be troublesome because it is currently **not reversible**. If you have further questions about proper usage of `alias`, please contact us either through our app or in the comments section below.

<!--
## 4. Consider: when you are using several of our integrations.

KISSmetrics lets you incorporate data from several data sources--from integrations with multiple platforms and from libraries in several languages. They may not all use the same Customer ID, so you may need to find the common links between these sources. That way, you could script a way to alias all the relevant IDs together, ensuring your customer data is not fragmented.
-->

[section1]: #section-1
[section2]: #section-2
[section2.1]: #section-2.1
[section2.2]: #section-2.2
[section3]: #section-3
[section3.1]: #section-3.1
[section3.2]: #section-3.2
[section3.3]: #section-3.3
[section3.4]: #section-3.4

[common-methods]: /apis/common-methods
[js-ids]: /apis/javascript/javascript-identities.html
[url-api]: /apis/url.html
[id-from-form]: /apis/javascript/javascript-specific/#identifying-a-person-from-a-form-field