Premise of project :
Whether privacy/ad-blocking extensions can actually reduce the data collection happening from GenAI browser assistants or whether these assistants bake in data collection that gets through regardless.

All work will be done on a spare gmail account for privacy reasons and also to provide a clean baseline so extensions on my main account dont interfere with the research.

Ai agents choosen are:

.Sider
![](Pasted%20image%2020260925190044.png)

.Monica


![](Pasted%20image%2020260925190411.png)

.Merlin

![](Pasted%20image%2020260925190353.png)
Set up of all three is complete

Next we will install PlayWright which will allow us to write and run a script to carry out the test on each browser for us.Now i should probably explain what i mean by test.

The test in question:
I plan to use to claude to generate two HTML files which with fake data baked in for example a student ID card or customer credit card details.

Process of installing PlayWright:
1. First step is to make sure python is installed and running on my laptop.
![](Pasted%20image%2020260925190447.png)
2/Second step is to set up a isolated environment in my project folder as if something breaks we will be able to delete the venv folder and recreate it in seconds.
3/Install Playwright using bash commands and set it up
![](Pasted%20image%2020260925190514.png)
4/Next we will be installing mitmproxy.We will configure chrome to route its traffic through mitmproxy as this will let us log outgoing network requests and will let us observe if any of our fake sensitive data ended up in any of those requests.
![](Pasted%20image%2020260925190553.png)