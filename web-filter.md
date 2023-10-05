## Web Filter

Web filter acts when the **HTTP GET** method is used.  
Since domains and URLs are different entities (ex, [**domain**]http[:]//example[.]com/[**URL**]index.php?login=true) web filter can either block connections to te entire domain (HTTP Header Host: www.example.com) or specific URLs (/blog/hacking).  

Web-Filter + Flow-based inspection + Profile-based (**Security Profiles** > **Web Filter**):

- web filters are defined as security profiles and applied to policies
- Fortiguard categories
- Static URL
- Rating option

Web-Filter + Proxy-based inspection + Policy-based (**Policy & Objects** > **Security Policy**):

- URLs categories are defined directly under the firewall policy

Web-Filter + Proxy-based inspection (**Security Profiles** > **Web Filter**):

- web filters are defined as security profiles and applied to policies
- local categories
- remote categories
- search engines
- proxy options

Fortiguard category filter is a live service requiring an active subscription to work properly. When there is no subscription but Fortigate make requests, rating errors are raised which block traffic.  

FortiManager can act as a local Fortiguard downloading a local copy of the database.  

Execpt for policy-based mode, web filter can **Allow** or **Deny** traffic. Can present a **Warning** message to users which are offered the ppossibility to exit the website or skip the warning message and continue. Or can require users to **Authenticate**.  

When users are aked to authenticate with valid usernames and passwords in order to access the domain/URL belonging to the category for which the "authenticate" action was chosen, they must submit valid credentials or have already been authenticated with passive authentication methods.  

Authentication is required per category and expires after X time.

Web filter with proxy-based inspection and full SSL inspection allows administrator to enforce **safe-search** for supported search engines (such as Google, Bing, Yahoo) altering the URL (example, appending _&safe=true_), and allows adrministrators to peform content filtering.  

Content filtering is based on simple words, patterns, Perl regular expressions to which numerical value, or scores, are attached.  
The higher a single score, the more impact has in the overall content score which is compared to the threshold set in the web filter profile.  
When the content score is higher, Fortigate blocks the site.  

Web Filter follows this HTTP inspection order:

- local static URL
- Fortiguard category filtering
- Advanced filters (safe-search...)

Running the command `diagnose debug rating` a list of Fortiguard server to which is possible to connect to is returned along with the following information:

- **Weight**: default = (difference in time zone) x 10
- **RTT**: Return Trip Time
- **Flags**: `D` (IP returned from DNS) `I` (contract server contacted), `T` (being timed), `F` (failed)
- **TZ**: Server time zone
- **Fortiguard-request**: the number of requests sent by Fortigate to Fortiguard
- **Curr Lost**: current number of lost request to Fortiguard (in a row, resets to 0 when 1 packet succeeds)
- **Total Lost**: total number of lost Fortiguard requests