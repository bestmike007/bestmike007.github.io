---
title: Free Lightweight Groupware Solution for Small Business
author: mike
layout: post
permalink: /2012/09/free-lightweight-groupware-solution-for-small-business/
comments: true
categories:
  - IT
tags:
  - Free
  - Groupware
  - LDAP
  - lightweight
  - Small Business
---
It's been a long time since my last blog post. Recently, I&#8217;ve been working on our own company since spring. We are planning to sell novel and creative things targeted United States market. As the director of the IT department of our company, I&#8217;ve been considering how everyone in our company can co-operate. We&#8217;re planning to deploy several common groupware systems, e.g. the bug tracking system, the internal wiki system, etc. And we also need an active directory to provide credentials for these systems.

<!--more-->

## The Problem

Of course, the solution can be:

1.  Microsoft Exchange
2.  VMWare Zimbra
3.  Open X-change

But Microsoft Exchange is too expensive for most small business companies to afford. We&#8217;ve been using Zimbra Community for a little while. It has most of the features we need including mail, calendar, ldap, etc. It&#8217;s really simple and easy to use, however, it has several disadvantages:

1.  Additional work for hardware maintenance
2.  Risks of data loss caused by hardware failure
3.  Not-quite-familiar user interfaces
4.  No mobile support with community edition

These advantages coalesced to force me reconsider another long-term solution. And this solution comes from the very simple principles resulting from trying to get rid of those disadvantages listed above. But how?

## The Solution

First and foremost, pick up your most favorite email service provider, e.g. Gmail (with calendar and mobile access). Then create new accounts for everyone in your company. And enable IMAP for all these accounts.

Secondly, develop the ldap server. Don&#8217;t panic, it&#8217;s really simple with <a title="OpenDJ LDAP SDK" href="http://opendj.forgerock.org/opendj-ldap-sdk/" target="_blank">OpenDJ LDAP SDK</a>. It&#8217;s a simple delegate server which authenticate the users with the Gmail IMAP server. Just create a new Java application with the following code:

<pre><code class="language-java">/**
 *
 * @author Mike
 */
public class Server {
    /**
     * @param args the command line arguments
     */
     public static void main(String[] args) throws Exception {
        File csv = new File("address_biz.csv");
        System.out.println(csv.getAbsolutePath());
         final ServerConnectionFactory&lt;LDAPClientContext, Integer&gt; connectionHandler
            = Connections.newServerConnectionFactory(new IMAPBackend());
         final LDAPListenerOptions options = new LDAPListenerOptions().setBacklog(4096);
         LDAPListener listener = null;
         try {
             listener = new LDAPListener("0.0.0.0", 389, connectionHandler, options);
             System.out.println("Running the ldap server...");
             synchronized (listener) {
                 listener.wait();
             }
         } catch (Exception ex) {
             ex.printStackTrace(System.err);
         } finally {
             if (listener != null) {
                 listener.close();
             }
         }
     }
}</code></pre>

IMAPBackend is the LDAP request handler:

<pre><code class="language-java">public class IMAPBackend implements RequestHandler&lt;RequestContext&gt; { ... }</code></pre>

And we only implement 2 of those methods, the first one is the bind method:

<pre><code class="language-java">@Override
public void handleBind(RequestContext c, int i, BindRequest br,
        IntermediateResponseHandler irh, ResultHandler rh) {
    GenericBindRequest request = (GenericBindRequest) br;
    String dn = br.getName();
    System.out.println("Binding: " + dn);
    String pass = new String(request.getAuthenticationValue());
    String uid = "";
    for (String s : dn.split(",")) {
        s = s.trim();
        if (s.startsWith("uid=")) {
            uid = s.substring(4);
        }
    }

    Properties props = System.getProperties();
    props.setProperty("mail.store.protocol", "imaps");
    try {
        Session session = Session.getDefaultInstance(props, null);
        Store store = session.getStore("imaps");
        store.connect("imap.gmail.com", 993, uid + "@gmail.com", pass);
        store.close();
        rh.handleResult(Responses.newBindResult(ResultCode.SUCCESS));
        System.out.println("Bind " + uid + " success.");
    } catch (Exception ex) {
        //ex.printStackTrace(System.err);
        rh.handleResult(Responses.newBindResult(ResultCode.INVALID_CREDENTIALS));
        System.out.println("Bind " + uid + " fail.");
    }
}</code></pre>

The second one is the search method, in which you need to maintain an address book for your company:

<pre><code class="language-java">@Override
public void handleSearch(RequestContext c, SearchRequest sr,
        IntermediateResponseHandler irh, SearchResultHandler srh) {
    System.out.println("Searching: " + sr.getFilter().toString());
    for (Entry e : load()) {
        if (sr.getFilter().matches(e).toBoolean()) {
            System.out.println("Search matched: " + e.getName());
            srh.handleEntry(Responses.newSearchResultEntry(e));
        }
    }
    srh.handleResult(Responses.newResult(ResultCode.SUCCESS));
}</code></pre>

And you should implement the load function, for example the following code load the address book from a csv file:

<pre><code class="language-java">private static List load() {
    File csv = new File("address.csv");
    if (!csv.exists()) {
        return Collections.emptyList();
    }
    Scanner scanner = null;
    try {
        scanner = new Scanner(new InputStreamReader(new FileInputStream(csv)));
        scanner.nextLine();
        List list = new ArrayList();
        while (scanner.hasNextLine()) {
            String attrs[] = scanner.nextLine().split("\",\"");
            String name = attrs[0].substring(1);
            String email = attrs[1];
            String alias[] = attrs[2].split(",");
            String uid = email.substring(0, email.indexOf("@"));
            String ou = attrs[3];
            if (ou.isEmpty()) {
                continue;
            }
            Entry e = new LinkedHashMapEntry();
            e.setName(DN.valueOf(String.format("uid=%s,ou=people,dc=gmail,dc=com", uid)));
            e.addAttribute("uid", uid);
            e.addAttribute("cn", name);
            e.addAttribute("email", email);
            e.addAttribute("mail", email);
            e.addAttribute("ou", "people");
            list.add(e);
            for (String a : alias) {
                if (a.isEmpty()) {
                    continue;
                }
                uid = a.substring(0, a.indexOf("@"));
                e = new LinkedHashMapEntry();
                e.setName(DN.valueOf(String.format("uid=%s,ou=people,dc=gmail,dc=com", uid)));
                e.addAttribute("uid", uid);
                e.addAttribute("cn", name);
                e.addAttribute("email", a);
                e.addAttribute("mail", a);
                e.addAttribute("ou", "people");
                list.add(e);
            }
        }
        scanner.close();
        return list;
    } catch (Exception ex) {
        ex.printStackTrace(System.err);
        return Collections.emptyList();
    } finally {
        if (scanner != null) {
            scanner.close();
        }
    }
}</code></pre>

And of course, you can implement your own search function with your preferred way to access your company address book. And that&#8217;s all for this ldap server.

At last, deploy the ldap server along with all the other groupware systems, e.g. MantisBT, mediawiki, etc.

Here is an example for the MantisBT configuration:

<pre><code class="language-php">$g_allow_signup = OFF;
$g_login_method = LDAP;
$g_ldap_server = 'ldap.example.com';
$g_ldap_root_dn = 'dc=gmail,dc=com';
$g_ldap_protocol_version = 3;
$g_ldap_bind_dn = 'uid=ldap.example.com,dc=gmail,dc=com';
$g_ldap_bind_passwd = 'internal_s3cret';
$g_use_ldap_email = ON;
$g_ldap_realname = 'cn';
$g_use_ldap_realname = ON;</code></pre>

And the example configuration for media wiki:

<pre><code class="language-php">$wgGroupPermissions['*']['createaccount'] = false;
$wgGroupPermissions['*']['edit'] = false;
$wgGroupPermissions['*']['read'] = false;
$wgGroupPermissions['*']['createpage'] = false;
$wgGroupPermissions['*']['createtalk'] = false;
$wgGroupPermissions['*']['writeapi'] = false;

# LDAP
require_once( dirname( __FILE__ ) . "/extensions/LdapAuthentication/LdapAuthentication.php" );
$wgAuth = new LdapAuthenticationPlugin();

$wgLDAPDomainNames = array("example.com");
$wgLDAPDebug = 3; $wgDebugLogGroups["ldap"] = "/tmp/ldap.log" ;
$wgLDAPBaseDNs = array("example.com" =&gt; "dc=gmail,dc=com");
$wgLDAPServerNames = array("example.com" =&gt; "ldap.example.com");
$wgLDAPSearchAttributes = array('example.com' =&gt; "uid");
$wgLDAPEncryptionType = array("example.com" =&gt; "clear");
$wgLDAPProxyAgent = array("example.com" =&gt; "uid=ldap.example.com,dc=gmail,dc=com");
$wgLDAPProxyAgentPassword = array("example.com" =&gt; "internal_s3cret");</code></pre>

##  Conclusion

This is a really simple solution which suit well for our company. We&#8217;re now using tencent enterprise email solution free edition with our own domain name. And we&#8217;re going to extend this whole system and make it robust for the long run. I sincerely hope that this can also help your business and please feel free to contact me (i &#8216;at&#8217; bestmike007.com). Thank you!