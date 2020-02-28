# Fraud Case Management Demo

What's covered in the demo
==========================

1.  Steps to build the Fraud Case Management BPM Process

2.  Using Guided Decision Tables

3.  Using subprocesses

4.  Using Timers

5.  Using WorkItems to generate emails

6.  Using WorkItems to Call Rest APIs

7.  Using a spring boot service to generate a PDF

Demo Overview
=============

The demo addresses a business process where a bank wants to be able to contact their customers once a fraud case is detected and then verify the case with the customer.

If the customer confirms that this is indeed a fraud transaction the system should then suspend the Credit Card. If the agent is unable to contact the customer the system should attempt to notify the customer via email (telegram or SMS in the future). The system will then wait for the customer to try and contact the bank for a specific period of time to validate the case if that doesn't happen the system will suspend the Credit Card and notify the customer via email and mail.

An exception to that flow would be if the case is deemed high risk, then the system will immediately suspend the Credit Card and notify the customer.

Business Process - Fraud Case Management
========================================

![](https://lh6.googleusercontent.com/ik18Oaq4iw0m9FgDMdmC8bFTf7uHC_MEVxPVtocWps93kBmFS5kMILfiyTrSAaVQvmlN1FGyUr4dJCSq4C35VETf8owfBSbbI5I7P0hXAaH0c45duO5z5gHoQwDc0zb1BQnOxwbk)

1 - Risk Evaluation
-------------------

The first step of the flow evaluates the risk of the CC transaction based on the customer segment and the amount. The transaction is deemed high risk if it satisfies one of the following conditions:

1.  Customer Segment is Silver and the amount is higher than 500 AUD

2.  Customer Segment is Gold and the amount is higher than 1000 AUD

3.  Customer Segment is Platinum and the amount is higher than 2000 AUD

This is implemented in the solution as a Guided Decision Table

![](https://lh5.googleusercontent.com/dCOgCJVaAGG4Jz-N_-h_OvJw97gCiJMKBpXJT84bhvX_3b2XvIdG4DyETp_bL7glUs-Gr6_8HE36thP7ZgOVQ9urgZTHYXALEXUS5XPNZecdRxYaadBSeALFVAMAPulIXLwpeq34)

Note: The rule flow group needs to be set to evaluate-risk-group-flow to link the business process to the rule

![](https://lh5.googleusercontent.com/bhNLZWK1id7WGhLhQl-4L0JFJ8o-ftL8ZnzUl5gPRHl0piHGn7LEmXz6D8ACD2EOsbaLW4kp8rFXY4VsMXt4VrnpMT9u7ZZzPmsHSgVWpNTFSstNKfuUqmbEI1dR98KB9Nvgequ3)

2 - Suspend Credit Card subprocess
----------------------------------

![](https://lh3.googleusercontent.com/3RI1lWvNZradksv-CvWf7Tu7ooT_iSzwGKoD3G19MY9YrTggPfkPovE49xgrv0NaiTHm2bLhk0QSzL4xa1u0N_7WhFHNmUdrDGMWfkXhezWssPEfKOMd-4BxMUVKrQHCVRW0QLHR)

The suspend Credit Card step is implemented as a subprocess since it is going to be used twice in our flow and it involves three steps

1.  Rest call to suspend the Credit Card

2.  Generate an email

3.  Rest Call to generate a PDF

Note: For this step, we need to create two Work Item Handlers as follows:

![](https://lh3.googleusercontent.com/O_4VXyE2ikkB9mFeSHJLBdU9v3yqli3ui7nEgfr_wL64bF-cVgMCYgb-xbyz-9SIA23cwJZqvJ_zdwiqP0oPKwkG7vdeuA_i-Ja-WY91_E_6iQl54X1YjhgLSW3e6upnOY-O5l0-)![](https://lh3.googleusercontent.com/5IcUoQCTM2K9JZyy1Oqcfac8wfwzzBSZOydGFRXn9YToFV4J_sojEmBdPYtKPKWfUGIClf4bbfz24mGk2bM7y8ZhKw7emcsCa6tsFThug29IcmYkyFlfCMuBZ76OcUEMDxdBBVNz)

3 - Contact Customer User Task
------------------------------

If the transaction is not evaluated as high risk, the case will be then assigned to an agent who will attempt to contact the customer to validate the case. The agent will then update the case with the result or that he/she failed to contact the customer.

4 - Post Contact Subprocess 
----------------------------

![](https://lh3.googleusercontent.com/gTlm_J3qUT_708qXOCFxlQLj-0qGMCwUVCafEUVk2YLhK1wI_eUZQreZ3P4dLsxX__0zSZX_g_8mpsKGKC5ZAXNs2Ad2-jkRZ2yFyhoIbSE3ijdDpQyW6o0kyTQN8_EnCvKvlVSo)

Once contact is made the flow check if the isFraud variable is set to true. If it is a fraud case the process will again trigger the Suspend Credit Card subprocess otherwise the system will log that contact was made and the customer confirmed that this is not a fraud case

5 - Wait for customer contact timer
-----------------------------------

Using the ISO-8601 date format, the time is configured to fire once after 10 seconds. The configuration looks like this 

![](https://lh5.googleusercontent.com/lobyNfDk6nAJIqtPzyevPNvK43JrCOXhwHnZzQkOndYNYAZb7YgVBLPyedg70HgeY9Gw48Xn6LNXdmosw57ziUGOxGS0LHmkFfqUukJKLADDFhaHBDhhBoEohzOjC_MpTbY1ytzO)

6- Email Work Item Handler (WIH)

The Email WIH is configured via four parameters, this Body parameter is an example on how to bind parmaters values with the input the Email WIH

![](https://lh6.googleusercontent.com/JSrUO6jB-qYFUVOYQ8UjIRWa831P-zO-3XF4nmmiw-jfoVOD3jb6fuIEMBW_cXTIYgZWiSRRTqqCL4XLYiQR9pWmxPWt6mbrXTnKq7KcB4HiV6UJda89TEsAV98QAHAyxIKAs2Ts)

Fraud Case Services
===================

This module is a set of web services that complement our demo

1 Get Fraud Case
----------------

This web service does not require input and returns the details of the fraud case being investigated. Sample output

![](https://lh3.googleusercontent.com/BXzYWdEJw4mtU6Ow6MbB2b_tTN03IZYYmTnFLiNnu8VScQd7neZ3phHxgCcTLUxKmM09XO6VB6vYbyCGdri2V9sOBjfwBelz-OniK4vjMAHqTQmoWFT0xkdt-q2bu75pZ3QeF12b)

Note: the Fraud Card details are loaded in the init method, loading the data into the system is outside the scope of this demo

2 Suspend Card
--------------

This web service will change the status of the credit card loaded in context to false. For simplicity, it does not require and input will only return a response OK

3 Generate PDF
--------------

This web service generates the PDF document that will be sent to the customer's printing engine to be printed and then mailed to the customer's address. The demo uses the Apache PDFBox libraries to generate the PDF.

SMTP Server 
============

The demo uses the Fake SMTP server (<http://nilhcem.com/FakeSMTP/>) to mimic an email server for the purpose of this demo.

Steps to create the demo
========================

This is an overview of the demo compoenets

![](https://lh5.googleusercontent.com/c0lPO_18rFjtDxXBASkL-WFV9ZWdqAFu6cwVXUZrsV-HZB52I1RZMrI3o1ITwN4eign_t20pEKuXo6rrYJWwQIHUiUVWjZR3UMC0iTJ30gsC-Rh2iw2oYUK-6vkU_eBf4EgrNbnQ)

1\. Install Red Hat PAM using the standard installer
----------------------------------------------------

Note: This demo was built on Red Hat EAP, however, it could be easily ported and run on Red Hat OpenShift

2\. Import this repo into Project Space and start the cc_fraudmgmt_process process
----------------------------------------------------------------------------------

3\. Build and compile Fraud Case Services package and move it to your Red Hat EAP deploy folder. 
-------------------------------------------------------------------------------------------------

Note: The server.port property is set to 9090

4\. Prepare the SMTP Server
---------------------------

Download the Fake SMTP server (<http://nilhcem.com/FakeSMTP/>). Go to the folder where the server is downloaded and start the server by running the script startServer.sh

You will then be immediately greeted by the server console, now click the Start server button.

![](https://lh4.googleusercontent.com/W0Ao8G9I7haKHhHrUlV8Bo84-Hf20MadTkjnu8ODQaMuppV5MSajZjdn1jVhsqzCy_Rv1arE65eT_PXJSL5qyzwQJC9G-zeC4sH7QOe2f1lMyaKX0hp6XvRISGh6MaVMhC8zkSuE)

Demo Script 
============

Scenario 1: High-Risk Transaction
---------------------------------

Show that the credit card status is active by calling getFraudCase service (using Postman for example)

You should get the following result

![](https://lh6.googleusercontent.com/PV6vNM4cRvTSmSqzU4knE-YdSVZhWm0598jh0XeH9ZkmuJ9Pcrbw_zaCNaMN93XYaD4CeW5LX41kFJxcv4pkosUxk7IsyCepnMhBxiI6dAM1WnH0QR6QAmkGcNAzev6ZGpsT2aHk)

Now start the business process and provide the following input:

-   Customer Segment: Silver

-   Credit Card: 4501242464011473

-   Fraud Case ID: 101

-   Account Number: 17586733

-   Amount: 1000

-   Transaction Details: Transaction for 1000 AUD in Bangkok

This will set the FraudCase.isHighRisk variable to true which will then trigger the suspend credit card subprocess.

Now if you call the getFraudCase service one more time, the creditCardActive flag will be set to false

![](https://lh5.googleusercontent.com/fXGmazcqOwX8f6QNrJXzGVqx1NZr9e_n18J4faQmdRONjPfqEPh50HWyrQ7PZq5qpvyKunF0IRCFlNgA85oZxMLSls4qv8PQIC27RUo0CzVmeJkX3xqzCJFqvOqviKmVzhbMu9bl)

You will also notice a new mail in your SMTP Server

![](https://lh5.googleusercontent.com/v0DROZPi8ZkqQMquhhWjhkfBmWGR0F3raBydrv8K974ffvVdN873lzbkc0zYHV8sleQfuQtpK19f1XHE9IgM2T7YjrZ2uqM3ko41PEj0Sg_i7Dun01XB9U11j6bzL2fsPUI-YaRH)

![](https://lh6.googleusercontent.com/ls9x8iqZEGdC7spzndCxpHCwOX1bkFREK5AWyGNggrA6qB7nZ7gdCYWmrxJLU2thjLcmX6W4658wr6YhL5vna-W1amU4RBFxSogpryRSjYOQWvWNG0jv5FPMgpvINwcPRVdUuycX)

And finally, a PDF will be generated which looks like this

![](https://lh5.googleusercontent.com/6CNeaQV1qrZZrOW3hJ0pclYFTYgRzVFT9lZOAebB9ajIok1BPGqyMBcDRTeNWwKFAhz4Kv6O0w0mesZP42ImS5TgMJDnik9f3lWV58Ra842V8tTqT6EBdubJPDL6li2grU8NkuSe)

Scenario 2: Low-Risk, Contact Success, and Real Fraud Case
----------------------------------------------------------

In this scenario, we do not want to trigger the high-risk flow so we will provide the following input, you can simply reduce the amount or use a higher customer segment

start the business process and provide the following input:

-   Customer Segment: Silver

-   Credit Card: 4501242464011473

-   Fraud Case ID: 101

-   Account Number: 17586733

-   Amount: 400

-   Transaction Details: Transaction for 400 AUD in Bangkok

Once the process is triggered, the flow will move to the Contact Customer User Task this time and a task will be waiting in the user's Task Inbox. In this scenario, the agent is going to check both the IsFraud and Contact Made checkboxes

![](https://lh6.googleusercontent.com/nbtCUi4puy9iTtbPbuQkyZUULgfVKeQ3V16iluBzBPvOYRPxc_rKU6RelLu9ntOMoheUJg-L94-N51BJwcQHth1y1bWnS1QdCsiN3ObkCPCI6hn5PLhBGRVpet0CEgrVgTDLGTeu)

Once the agent clicks complete, this will trigger the post contact made sub process, which will in turn call the suspend subprocess similar to scenario 1

Scenario 3: Low-Risk and Contact Failed, Real Fraud Case
--------------------------------------------------------

This scenario will follow the same flow as scenario two unpt the point where a case is assigned to an agent. At this point provide the same input, but do not flag the Contact Made checkbox. Once the agent clicks complete an email will be triggered notifying the customer, then a timer will be triggered to wait for the customer to make contact. Once the timer expires, the customer the flow will trigger the Post Contact Made sub process similar to scenario two,

Future Work
===========

1 Send a telegram message
-------------------------

Use Fuse Online to integrate with Telegram

2 Build a User Interface
------------------------

Use Entando to build a better UI rather than using the default PAM forms

3 Migrate to Quarkus
--------------------

Migrate the Spring Boot services to Quarkus
