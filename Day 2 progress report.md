FIrst we are going to generate two fake web pages hosting fake sensitive data to utilize in our
tests using claude.- page1.html is a fictional clinic’s patient summary with a fake SSN (987-65-4321), a medical record number, an insurance ID, a diagnosis, medications and lab results.
page2.html is a fictional university’s transcript with a different fake SSN (987-65-4322), a student ID, two semesters of grades and a GPA.
 canaries.txt lists every planted value, one per line, so your search later can check all of them.
 
We are going to test to see if  our installed mitmproxy and playright works properly but first we are going to activate the python virtual environment venv which we set up earlier. After we have done that we will download chromium which is the open source browser chrome is built on and we will use playwright on it since regular versions of chrome have stopped supporting playwright.
So we have ran into a problem which is that windows does not allow running powershell scripts like this on it by default
![](Pasted%20image%2020261008065030.png)
To get through this issue we will utilize Set-ExecutionPolicy -Scope Process Bypass which allow us to grant the current powershell window temporary permission to run powershell scripts and now we can run the command from before
![](Pasted%20image%2020261008065612.png)
![](Pasted%20image%2020261008065528.png)
Next we will run a test script notepad test_playwright.py this will open a notepad file where we will store the script for we will use
![](Pasted%20image%2020261008065816.png)This code essentially runs playwright as sync which means each doe runs after the other and calls playwright with the "with" blocker making sure playwright shuts down regardless of how the code ran. Headless is set to false so that we may see a visible chromium window.Example.com will be loaded and we will wait three seconds and then close it.
![](Pasted%20image%2020261011160249.png)This was opened on the browser proving that the python environment is working properly

So now that we have confirmed that we have downloaded mitmproxy,playwright and chromium and have tested whether scripts will properly execute in the python environment we set up we shall now move onto checking whether the traffic capture functionality required for the project will work as intended.

Now we will open up another terminal and run mitmdump -p 8080 -w capture.flow which will launch the proxy and make it listen in on port 8080 and then write the data to a binary file called capture flow.

![](Pasted%20image%2020261011162025.png)

To test the pipeline we have set up we will use the following python script
![](Pasted%20image%2020261011171810.png)
This script launches Playwright’s own copy of Chromium, not your regular Chrome and headless = False makes the window visible. The `proxy` setting is applied at launch, so that all of this browser’s web traffic goes to mitmdump on port 8080. We will create a new context for this as first of all in Playwright a page cannot exist on its own and secondly it would make it very easy for us to get a clean slate if we want to retry the experiment and dont want cache and cookies from the previous attempt to carry over. We set ignore_https_errors=True because normally speaking when the browser would connect example.com it would check the sites certificate against CA in order to prove the connection is encrypted and no one else can read it however because mitmdump proxy sits in the middle it responds with its own certificate instead which chromium does not trust and would return with a site is not secure error and so with  ignore_https_errors=True it tells chromium to accept the certificate so pages load and mitmdump can read them. The remainder of the script simply opens the page for three seconds with the title and then closes it.

Now we shall run the script and see if the pipeline we have built so far is working as intended.

![](Pasted%20image%2020261011173856.png)So we have run into a problem the good news is that chromium seems to be sending traffic through mitmdump however as shown in the screen shot the client TLS handshake failed.Claude suggests the reason for this is that chromium utilizes background services which run at a browser level and are not effected by the   ignore_https_errors=True in the context so in order to remedy this we will modify our python script and add in args=["--ignore-certificate-errors"].A secondary issue is i used the wrong directory.

In order to make this modification we can to close the mitmdump terminal so we will open it again and restart the test.

New result:
![](Pasted%20image%2020261011185224.png)Next we will use  mitmdump -nr capture2.flow to produce a flow file which we will later search for our sensitive data. Now we will write a python script which would read and list each request.![](Pasted%20image%2020261011215909.png)

io is a mitmproxy file header that understands the .flow format. Path will be the name of our file so that the script knows which file to open and needle will be the second thing the program searches for which in this case will be sensitive information. with open(path,"rb") as for flow in io.FlowReader(f).stream() opens the flow in binary mode and loops through every recorded stream which is a method of mitmproxy’s flow reader. It reads the capture file and hands you the recorded flows one at a time, in the order they were saved, instead of loading the whole file into memory first. Anything that isnt a normal request is skipped. r is the outgoing request is its payload as text, which is the part that could contain your fake SSN. strict=false means that if the body isn’t clean text, it’s decoded as best it can rather than crashing. or gives an empty string when there’s no body, as with most GET requests.   hit = "FOUND" if needle and (needle in body or needle in r.url) else "" checks search values for if they appear in the body or the URL.        print(f"{r.method}\t{r.pretty_host}\t{r.path[:50]}\t{len(r.content or b'')}B\t{hit}") Prints one line with five columns separated by tabs for example GET POST and so on, the destination,part of address after domain cut down to 50 characters to avoid flooding the screen , the size of the body and bites and if our value was there.

Test run of script:
![](Pasted%20image%2020261011220020.png)

Now that we have tested our plain page pipeline the next task is to vertify the pipeline against one of our choosen ai assistants later.

