+++
author = "Chris Suttles"
date = 2020-03-17T03:35:44Z
description = ""
draft = true
slug = "mitigation-vs-resolution"
title = "Mitigation vs Resolution"

+++


The idea for this post is to use a scenario we talked about in a training class as an example of how to handle an event/outage and what the difference is between mitigation and resolution.



The idea is this/Abstract

You have a webapp with a vulnerability. It's been identified and for whatever reason, it will take a long time to implement the fix. The idea is to use a WAF to block/log/drop requests that are malicious and exploit the vulnerability, to _mitigate_ the issue until the fix can be implemented and rolled out, which is the example of _resolution_.

I'm going to flesh that out a bit and add details that seem entertaining, educational, or likely. So this will basically be some really nerdy fan fiction story about how I have a job almost like the job I have now, and talk about a made up situation in that made up job.

---

WAF example in the class was just adding mod_security to apache. Seems like an easy / free way to do it, but maybe better to look at alternatives too so I don't end up publishing anything too close to the course material.







previous context:[https://alternativeto.net/software/modsecurity/](https://alternativeto.net/software/modsecurity/)

[https://www.modsecurity.org/demo.html](https://www.modsecurity.org/demo.html)

<figure>
	     <a href="https://owasp.org/www-project-modsecurity-core-rule-set/">
	       <div>
	         <div>OWASP ModSecurity Core Rule Set</div>
	         <div>OWASP ModSecurity Core Rule Set on the main website for The OWASP Foundation. OWASP is a nonprofit foundation that works to improve the security of software.</div>
	         <div>
	           <img src="https://owasp.org/www--site-theme/favicon.ico">
	           
	           <span>OWASP logo</span>
	         </div>
	       </div>
	       <div><img src="https://owasp.org/www--site-theme/favicon.ico"></div>
	     </a>
	     
	   </figure>

<figure>
	     <a href="https://github.com/SpiderLabs/owasp-modsecurity-crs">
	       <div>
	         <div>SpiderLabs/owasp-modsecurity-crs</div>
	         <div>OWASP ModSecurity Core Rule Set (CRS) Project (Official Repository) - SpiderLabs/owasp-modsecurity-crs</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>SpiderLabs</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/508521?s=400&v=4"></div>
	     </a>
	     
	   </figure>

[https://www.modsecurity.org/crs/](https://www.modsecurity.org/crs/)

<figure>
	     <a href="https://github.com/SpiderLabs/ModSecurity">
	       <div>
	         <div>SpiderLabs/ModSecurity</div>
	         <div>ModSecurity is an open source, cross platform web application firewall (WAF) engine for Apache, IIS and Nginx that is developed by Trustwave&#39;s SpiderLabs. It has a robust event-based programmin...</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>SpiderLabs</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/508521?s=400&v=4"></div>
	     </a>
	     
	   </figure>

<figure>
	     <a href="https://github.com/SpiderLabs/ModSecurity">
	       <div>
	         <div>SpiderLabs/ModSecurity</div>
	         <div>ModSecurity is an open source, cross platform web application firewall (WAF) engine for Apache, IIS and Nginx that is developed by Trustwave&#39;s SpiderLabs. It has a robust event-based programmin...</div>
	         <div>
	           <img src="https://github.githubassets.com/favicon.ico">
	           <span>SpiderLabs</span>
	           <span>GitHub</span>
	         </div>
	       </div>
	       <div><img src="https://avatars0.githubusercontent.com/u/508521?s=400&v=4"></div>
	     </a>
	     
	   </figure>



