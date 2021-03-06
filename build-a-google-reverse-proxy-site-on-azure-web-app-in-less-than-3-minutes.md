<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Recently I live in China, where many popular websites, such as Google and Facebook, are not accessible. So I write this article to share the easiest way to make these sites accessible by using Microsoft Azure. I'll use Google as the example.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">In this article, I will first explain why I choose to use Azure Web App, which is followed by the step-by-step guidance. And finally, the article ends with explanation about the configurations indetail.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-size: large;font-weight: bold">There Are Many Ways to Make These Sites Accessible</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">1. Set up a </span><a href="http://blogs.msdn.com/b/lighthouse/archive/2013/07/30/how-deploy-sstp-and-l2tp-vpn-in-windows-azure-windows-server-2012.aspx"><span style="font-family: Calibri;font-size: 11pt">VPN by using Azure Virtual Machine</span></a><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">.</span></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">2. Create an </span><a href="https://azure.microsoft.com/en-us/documentation/articles/cdn-create-new-endpoint/"><span style="font-family: Calibri;font-size: 11pt">Azure CDN</span></a><span style="font-family: Calibri;font-size: 11pt"> with Google as the origin server.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">3. Host a REST service, such as </span><a href="http://stackoverflow.com/questions/25145098/how-to-set-up-web-api-routing-for-a-proxy-controller"><span style="font-family: Calibri;font-size: 11pt">Web API</span></a><span style="font-family: Calibri;font-size: 11pt">, on Web Role or Web App as a relay.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">4. Use </span><a href="http://www.iis.net/learn/extensions/url-rewrite-module/reverse-proxy-with-url-rewrite-v2-and-application-request-routing"><span style="font-family: Calibri;font-size: 11pt">ARR and URL Rewrite modules</span></a><span style="font-family: Calibri;font-size: 11pt"> in IIS to set up a reverse proxy service in a virtual machine.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">5. Use </span><a href="http://www.iis.net/learn/extensions/url-rewrite-module/reverse-proxy-with-url-rewrite-v2-and-application-request-routing"><span style="font-family: Calibri;font-size: 11pt">ARR and URL Rewrite modules</span></a><span style="font-family: Calibri;font-size: 11pt"> to build a reverse proxy service Web App.</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Approach 1 is the ultimate solution, which allows you to access any public website (be aware of DNS cache poisoning, though). However it requires you to create a virtual machine and purchase a certificate if you want to share it with others. Not everyone likes it.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Approach 2 is simple but it doesn't work for server redirection or references to the backend server with absolute URL.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Approach 3 requires to write some code, which can be tedious.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Approach 4 also requires to create a virtual machine, which is more expensive.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Approach 5 is the most lightweight approach. So I will share this approach in detail. How lightweight it is? Well, nothing other than a web browser is required, and you don't even need an Azure subscription to go through the steps below.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-size: large;font-weight: bold">When Do You Want to Build A Reverse Proxy on Azure Web App?</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">1. The backend website is publicly accessible from Azure, but is not accessible in your region.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">2. You can access Microsoft Azure global instance.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">3. You have an Azure subscription (global instance) or you are willing to create one.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">4. You probably want to share the site with others.</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-size: large;font-weight: bold">Step-by-step Guidance</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">(If you already have an Azure subscription, and know <a href="https://azure.microsoft.com/en-us/documentation/articles/web-sites-dotnet-get-started/">how to create a Web App</a>, you can go to step 3 directly.)</p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">1. Go to </span><a href="https://tryappservice.azure.com"><span style="font-family: Calibri;font-size: 11pt">https://tryappservice.azure.com</span></a><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal"> in your web browser. In</span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: bold"> Select your app</span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal"> section, choose </span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: bold">Web App</span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">, and click </span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: bold">Next</span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal"> button.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal"><img src="images/8117.Select_your_app.png" alt="" border="0" /></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">2. In </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Select a template and create your Web App</span><span style="font-family: Calibri;font-size: 11pt"> section, choose </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Empty Site</span><span style="font-family: Calibri;font-size: 11pt">, and click </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Create</span><span style="font-family: Calibri;font-size: 11pt"> button. When asked to log in, use your Microsoft Account.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/1777.Select_a_template_and_create_your_Web_App.png"><img src="images/1777.Select_a_template_and_create_your_Web_App.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">3. Now your web app is created. In my case, it is </span><a href="https://efe73b37-0ee0-4-231-b9ee.azurewebsites.net/"><span style="font-family: Calibri;font-size: 11pt">https://efe73b37-0ee0-4-231-b9ee.azurewebsites.net/</span></a><span style="font-family: Calibri;font-size: 11pt"> (it is a temporary one, and won't be available when you read this article). I will use "xyz" to replace the "efe73b37-0ee0-4-231-b9ee" part below. You will have a similar one which is different in the first part of the domain name. </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Don't visit your new site yet!</span><span style="font-family: Calibri;font-size: 11pt"> If you happen to do so, refer to steps a-d.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/0257.Your_web_app_has_been_created.png"><img src="images/0257.Your_web_app_has_been_created.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">4. Go to the SCM (</span><a href="http://blogs.msdn.com/b/benjaminperkins/archive/2014/03/24/using-kudu-with-windows-azure-web-sites.aspx"><span style="font-family: Calibri;font-size: 11pt">Kudu</span></a><span style="font-family: Calibri;font-size: 11pt">) site of your web app: </span><a href="https://xzy.scm.azurewebsites.net/"><span style="font-family: Calibri;font-size: 11pt">https://xzy.scm.azurewebsites.net/</span></a><span style="font-family: Calibri;font-size: 11pt">. Not that you need to insert "scm." before "azurewebsite.net". This is the administration site for your web app.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">5. Navigate to </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Debug console -</span><span style="font-family: Calibri;font-size: 11pt">&gt; </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">CMD</span><span style="font-family: Calibri;font-size: 11pt">.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/0358.Debug_console_CMD.png"><img src="images/0358.Debug_console_CMD.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">6. Navigate to </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">site</span><span style="font-family: Calibri;font-size: 11pt"> folder.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/4784.site.png"><img src="images/4784.site.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">7. In the CMD console, run </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">echo 1 &gt; applicationHost.xdt</span><span style="font-family: Calibri;font-size: 11pt"> to create the applicationHost.xdt file.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/2425.create_applicationHost.png"><img src="images/2425.create_applicationHost.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">8. You will see the file </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">applicationHost.xdt  </span><span style="font-family: Calibri;font-size: 11pt">created. Click the </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Edit</span><span style="font-family: Calibri;font-size: 11pt"> icon to edit it.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/4276.edit_applicationHost.png"><img src="images/4276.edit_applicationHost.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">9. Replace all contents in the editor with below text (also available </span><a href="https://github.com/zhiliangxu/reverse_proxy/blob/master/applicationHost.xdt"><span style="font-family: Calibri;font-size: 11pt">here</span></a><span style="font-family: Calibri;font-size: 11pt">). Then click the </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Save</span><span style="font-family: Calibri;font-size: 11pt"> button.</span></p>

<pre class="scroll"><code class="html"> &lt;?xml version="1.0"?&gt;
 &lt;configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform"&gt;
 &lt;system.webServer&gt;
 &lt;proxy xdt:Transform="InsertIfMissing" enabled="true" preserveHostHeader="false" reverseRewriteHostInResponseHeaders="false" /&gt;
 &lt;rewrite&gt;
 &lt;allowedServerVariables&gt;
 &lt;add name="HTTP_ACCEPT_ENCODING" xdt:Transform="InsertIfMissing" /&gt;
 &lt;add name="HTTP_X_ORIGINAL_HOST" xdt:Transform="InsertIfMissing" /&gt;
 &lt;/allowedServerVariables&gt;
 &lt;/rewrite&gt;
 &lt;/system.webServer&gt;
 &lt;/configuration&gt;</code></pre>
<span><strong>Note</strong> revised on 1/14/2018, changing <code class="html">Insert</code> to <code class="html">InsertIfMissing</code> for <code class="html">HTTP_ACCEPT_ENCODING</code> and <code class="html">HTTP_X_ORIGINAL_HOST</code> as advised by David Ebbo.</span>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/2210.save_applicationHost.png"><img src="images/2210.save_applicationHost.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">10. Navigate to </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">wwwroot</span><span style="font-family: Calibri;font-size: 11pt"> folder.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/8228.wwwroot.png"><img src="images/8228.wwwroot.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">11. In the CMD console, run </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">echo 1 &gt; web.config</span><span style="font-family: Calibri;font-size: 11pt"> to create the web.config file.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/1373.create_webconfig.png"><img src="images/1373.create_webconfig.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">12. You will see the file </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">web.config </span><span style="font-family: Calibri;font-size: 11pt">created. Click the </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Edit</span><span style="font-family: Calibri;font-size: 11pt"> icon to edit it.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/7870.edit_webconfig.png"><img src="images/7870.edit_webconfig.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">13. Replace all contents in the editor with below text (also available </span><a href="https://github.com/zhiliangxu/reverse_proxy/blob/master/web.config"><span style="font-family: Calibri;font-size: 11pt">here</span></a><span style="font-family: Calibri;font-size: 11pt">). Then click the </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Save</span><span style="font-family: Calibri;font-size: 11pt"> button.</span></p>

<pre class="scroll"><code class="html">&lt;configuration&gt;
 &lt;system.webServer&gt;
 &lt;httpErrors errorMode="Detailed" /&gt;
 &lt;rewrite&gt;
 &lt;rules&gt;
 &lt;rule name="ForceSSL" stopProcessing="true"&gt;
 &lt;match url="^(.*)" /&gt;
 &lt;conditions&gt;
 &lt;add input="{HTTPS}" pattern="^off$" ignoreCase="true" /&gt;
 &lt;/conditions&gt;
 &lt;action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" /&gt;
 &lt;/rule&gt;
 &lt;rule name="ProxyGStatic" stopProcessing="true"&gt;
 &lt;match url="^gstatic/(.*)" /&gt;
 &lt;action type="Rewrite" url="https://encrypted-tbn1.gstatic.com/{R:1}" /&gt;
 &lt;serverVariables&gt;
 &lt;set name="HTTP_ACCEPT_ENCODING" value="" /&gt;
 &lt;set name="HTTP_X_ORIGINAL_HOST" value="{HTTP_HOST}" /&gt;
 &lt;/serverVariables&gt;
 &lt;/rule&gt;
 &lt;rule name="ProxySSLGStatic" stopProcessing="true"&gt;
 &lt;match url="^sslgstatic/(.*)" /&gt;
 &lt;action type="Rewrite" url="https://ssl.gstatic.com/{R:1}" /&gt;
 &lt;serverVariables&gt;
 &lt;set name="HTTP_ACCEPT_ENCODING" value="" /&gt;
 &lt;set name="HTTP_X_ORIGINAL_HOST" value="{HTTP_HOST}" /&gt;
 &lt;/serverVariables&gt;
 &lt;/rule&gt;
 &lt;rule name="Proxy" stopProcessing="true"&gt;
 &lt;match url="(.*)" /&gt;
 &lt;action type="Rewrite" url="https://www.google.com.sg/{R:1}" /&gt;
 &lt;serverVariables&gt;
 &lt;set name="HTTP_ACCEPT_ENCODING" value="" /&gt;
 &lt;set name="HTTP_X_ORIGINAL_HOST" value="{HTTP_HOST}" /&gt;
 &lt;/serverVariables&gt;
 &lt;/rule&gt;
 &lt;/rules&gt;
 &lt;outboundRules&gt;
 &lt;preConditions&gt;
 &lt;preCondition name="IsHTML"&gt;
 &lt;add input="{RESPONSE_CONTENT_TYPE}" pattern="^text/html" /&gt;
 &lt;/preCondition&gt;
 &lt;preCondition name="IsJson"&gt;
 &lt;add input="{RESPONSE_CONTENT_TYPE}" pattern="^application/json" /&gt;
 &lt;/preCondition&gt;
 &lt;/preConditions&gt;
 &lt;rule name="ChangeReferencesToOriginalUrl" preCondition="IsHTML"&gt;
 &lt;match filterByTags="A, Area, Base, Form, Frame, Head, IFrame, Img, Input, Link, Script" pattern="^https://www.google.com.sg/(.*)" /&gt;
 &lt;action type="Rewrite" value="https://{HTTP_X_ORIGINAL_HOST}/{R:1}" /&gt;
 &lt;/rule&gt;
 &lt;rule name="ChangeReferencesToOriginalUrlInJson" patternSyntax="ExactMatch" preCondition="IsJson"&gt;
 &lt;match pattern="www.google.com.sg" /&gt;
 &lt;action type="Rewrite" value="{HTTP_X_ORIGINAL_HOST}" /&gt;
 &lt;/rule&gt;
 &lt;rule name="ChangeGStaticReferencesToOriginalUrl" preCondition="IsHTML"&gt;
 &lt;match filterByTags="A, Area, Base, Form, Frame, Head, IFrame, Img, Input, Link, Script" pattern="^https://encrypted-tbn[0-9].gstatic.com/(.*)" /&gt;
 &lt;action type="Rewrite" value="https://{HTTP_X_ORIGINAL_HOST}/gstatic/{R:1}" /&gt;
 &lt;/rule&gt;
 &lt;rule name="ChangeGStaticReferencesToOriginalUrlInJson" preCondition="IsJson"&gt;
 &lt;match pattern="encrypted-tbn[0-9]\.gstatic\.com" /&gt;
 &lt;action type="Rewrite" value="{HTTP_X_ORIGINAL_HOST}/gstatic" /&gt;
 &lt;/rule&gt;
 &lt;rule name="ChangeSSLGstaticReferencesToOriginalUrl" patternSyntax="ExactMatch" preCondition="IsHTML"&gt;
 &lt;match pattern="ssl.gstatic.com" /&gt;
 &lt;action type="Rewrite" value="{HTTP_X_ORIGINAL_HOST}/sslgstatic" /&gt;
 &lt;/rule&gt;
 &lt;rule name="RewriteBackendRelativeUrlsInRedirects" preCondition="IsHTML"&gt;
 &lt;match serverVariable="RESPONSE_LOCATION" pattern="^https://www.google.com.sg/(.*)" /&gt;
 &lt;action type="Rewrite" value="https://{HTTP_X_ORIGINAL_HOST}/{R:1}" /&gt;
 &lt;/rule&gt;
 &lt;/outboundRules&gt;
 &lt;/rewrite&gt;
 &lt;/system.webServer&gt;
 &lt;/configuration&gt;</code></pre>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/3733.save_webconfig.png"><img src="images/3733.save_webconfig.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">14. Now you can browse </span><a href="https://xyz.azurewebsites.net/"><span style="font-family: Calibri;font-size: 11pt">https://xyz.azurewebsites.net/</span></a><span style="font-family: Calibri;font-size: 11pt">. Voila! The main page of Google is displayed. </span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="images/8015.Google.png"><img src="images/8015.Google.png" alt="" border="0" /></a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">If you have visited the site before editing applicationHost.xdt file, you need to do the following steps to restart the web app, since applicationHost.xdt is only processed when the web app starts.</p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">a. Navigate to </span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: bold">Process Explorer</span><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal">.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt;font-style: normal;font-weight: normal"><a href="images/5432.Process_explorer.png"><img src="images/5432.Process_explorer.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">b. Press </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Properties..</span><span style="font-family: Calibri;font-size: 11pt"> button for the w3wp.exe (the one without "scm").</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/4621.Process_explorer_properties.png"><img src="images/4621.Process_explorer_properties.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">c. In the Properties pop up window, press the red </span><span style="font-family: Calibri;font-size: 11pt;font-weight: bold">Kill</span><span style="font-family: Calibri;font-size: 11pt"> button.</span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt"><a href="images/7888.kill.png"><img src="images/7888.kill.png" alt="" border="0" /></a></span></p>
<p style="margin-top: 0px;margin-bottom: 0px;vertical-align: middle"><span style="font-family: Calibri;font-size: 11pt">d. Now you can visit </span><a href="https://efe73b37-0ee0-4-231-b9ee.azurewebsites.net/"><span style="font-family: Calibri;font-size: 11pt">https://efe73b37-0ee0-4-231-b9ee.azurewebsites.net/</span></a><span style="font-family: Calibri;font-size: 11pt">. Refresh for several times (with Ctrl + F5 in IE to avoid content caching) if you still see the old page. The w3wp.exe process will be recreated automatically when you visit the site.</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-size: large;font-weight: bold">Explanations in Detail</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">The reverse proxy Web App works in the way illustrated as below.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="images/8468.BigPicture.png"><img src="images/8468.BigPicture.png" alt="" border="0" /></a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">User sends the request to your web app <a href="https://xyz.azurewebsites.net">https://xyz.azurewebsites.net</a>. When the request reaches your web app, the URL rewrite rules configured in your web app kick in. The request's URL is changed to <a href="https://www.google.com">https://www.google.com</a>, Accept-Encoding HTTP header is cleared, and then the modified request is forwarded to Google.
Google sends back the uncompressed response to your web app. Your web app scans the response and replaces references from <a href="https://www.google.com/*">https://www.google.com/*</a> to <a href="https://xyz.azurewebsites.net/*">https://xyz.azurewebsites.net/*</a>, as well as other minor changes, and then sends back the modified response to the user. The users see the response as if your web app hosts a clone of Google site.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold">Step 1-3:</span> A Web App created with <a href="https://tryappservice.azure.com/">Try Azure App Service</a> lasts for only one hour for you to play with. If you find this approach useful, why not start with a <a href="https://azure.microsoft.com/en-us/pricing/free-trial/">free Azure subscription</a> today?</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold">Step 7: ApplicationHost.xdt</span> is an Web App extension for you to transform the ApplicationHost.config file of your website. For more details, read <a href="https://azure.microsoft.com/en-us/documentation/articles/web-sites-transform-extend/">this article</a>.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold">Step 9:</span> The <span style="font-weight: bold">proxy </span>tag is to enable revere proxy functionality of <a href="http://www.iis.net/learn/extensions/url-rewrite-module/reverse-proxy-with-url-rewrite-v2-and-application-request-routing">Application Request Routing</a>. The <span style="font-weight: bold">allowedServerVariables </span>tag is to enable server variables to be set in URL Rewrite rules. I will discuss URL Rewrite rules in detail when explaining step 13. Without this <span style="font-weight: bold">allowedServerVariables </span>tag, you will get URL Write Module Error such as 'The server variable "HTTP_ACCEPT_ENCODING" is not allowed to be set. Add the server variable name to the allowed server variable list.'</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="images/8877.url_rewrite_module_error.png"><img src="images/8877.url_rewrite_module_error.png" alt="" border="0" /></a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold">Step 13: Web.config</span> file is used mainly to describe URL Rewrite rules. See <a href="http://www.iis.net/learn/extensions/url-rewrite-module/url-rewrite-module-20-configuration-reference">this article</a> for how to write rules for URL Rewrite 2.0.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-weight: bold">httpErrors</span> tag is to turn on debug information in case there is any URL Rewrite module error. The rules, except ForceSSL, in <span style="font-weight: bold">rules</span> (inbound rules) tag define how the web request is pre-processed before forwarded to Google.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ForceSSL</span> is to redirect HTTP traffic to HTTPS endpoint for your site.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ProxyGStatic</span> is to rewrite URL of <a href="https://xyz.azurewebsites.net/gstatic/*">https://xyz.azurewebsites.net/gstatic/*</a> to <a href="https://encrypted-tbn1.gstatic.com/*">https://encrypted-tbn1.gstatic.com/*</a>. This is used together with the outbound rules <span style="font-weight: bold">ChangeGStaticReferencesToOriginalUrl </span>and <span style="font-weight: bold">ChangeGStaticReferencesToOriginalUrlInJson</span>. <span style="font-weight: bold">ProxySSLGStatic</span> is for similar purpose.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">Proxy</span> is to rewrite URL of <a href="https://xyz.azurewebsites.net/*">https://xyz.azurewebsites.net/*</a> to <a href="https://www.google.com/*">https://www.google.com/*</a>. This is the key of the configurations. All other rules are just to improve user experience of the web app.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">The <span style="font-weight: bold">serverVariables</span> tags in inbound rules are used to clear the Accept-Encoding HTTP header and save the host name. It is important to clear the Accept-Encoding HTTP header, or otherwise you cannot execute outbound rules since the content returned by Google is compressed. I configured <span style="font-weight: bold">allowedServerVariables</span> in<span style="font-weight: bold"> applicationHost.xdt</span> so that these server variables can be set in URL Rewrite rules.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">The rules in <span style="font-weight: bold">outboundRules</span> tag defines how the web responses from Google are post-processed in the web app before sent back to user.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ChangeReferencesToOriginalUrl</span> is to change references (such as hyperlinks, images) to Google in the HTML responses to go through your web app.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ChangeReferencesToOriginalUrlInJson</span> is to change all references to Google in JSON responses to your web app.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ChangeGStaticReferencesToOriginalUrl</span> is to change references (such as hyperlinks, images) to <a href="https://encrypted-tbn[0-9].gstatic.com/*">https://encrypted-tbn[0-9].gstatic.com/*</a> in the HTML responses to <a href="https://xzy.azurewebsites.net/gstatic/*">https://xzy.azurewebsites.net/gstatic/*</a> virtual directory of your web app. It is supported by the <span style="font-weight: bold">ProxyGStatic </span>inbound rule.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ChangeGStaticReferencesToOriginalUrlInJson</span> is to change all references to <a href="https://encrypted-tbn[0-9].gstatic.com/*">https://encrypted-tbn[0-9].gstatic.com/*</a> in JSON responses to the <a href="https://xzy.azurewebsites.net/gstatic/">https://xzy.azurewebsites.net/gstatic/*</a> virtual directory of your web app. It is supported by the <span style="font-weight: bold">ProxyGStatic </span>inbound rule.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">ChangeSSLGstaticReferencesToOriginalUrl</span> is to change all references to ssl.gstatic.com to the <a href="https://xzy.azurewebsites.net/sslgstatic/*">https://xzy.azurewebsites.net/sslgstatic/*</a> virtual directory of your web app. It is supported by the <span style="font-weight: bold">ProxySSLGStatic </span>inbound rule.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">Rule <span style="font-weight: bold">RewriteBackendRelativeUrlsInRedirects</span> is to change location of 301/302 redirection to your web app if it is originally <a href="https://www.google.com/*">https://www.google.com/*</a>.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt">The two pre-conditions in <span style="font-weight: bold">preCondition</span> tag, <span style="font-weight: bold">IsHTML</span> and <span style="font-weight: bold">IsJson</span>, are used by the outbound rules to check if the response content type is HTML or JSON, respectively.</p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><span style="font-size: large;font-weight: bold">References</span></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="https://tomssl.com/2015/06/15/create-your-own-free-reverse-proxy-with-azure-web-apps/">https://tomssl.com/2015/06/15/create-your-own-free-reverse-proxy-with-azure-web-apps/</a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="http://ruslany.net/2014/05/using-azure-web-site-as-a-reverse-proxy/">http://ruslany.net/2014/05/using-azure-web-site-as-a-reverse-proxy/</a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="http://www.iis.net/learn/extensions/url-rewrite-module/url-rewrite-module-20-configuration-reference">http://www.iis.net/learn/extensions/url-rewrite-module/url-rewrite-module-20-configuration-reference</a></p>
<p style="margin: 0in;font-family: Calibri;font-size: 11pt"><a href="http://stackoverflow.com/questions/15926203/iis-as-a-reverse-proxy-compression-of-rewritten-response-from-backend-server">http://stackoverflow.com/questions/15926203/iis-as-a-reverse-proxy-compression-of-rewritten-response-from-backend-server</a></p>
