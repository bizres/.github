# Welcome to the "businessresponsibility.ch" project! 👋

<table border="0">
  <tr>
    <td>
    <a href="https://en.businessresponsibility.ch" target="_blank">
        <img src="https://static.wixstatic.com/media/96dfe9_4283a3265e03496298ee0c81559d8964~mv2.png/v1/fill/w_110,h_106,al_c,q_85,usm_0.66_1.00_0.01/logo_edited.webp" alt="https://en.businessresponsibility.ch/"
        width="256" height=""/>
    </a>
  <td align="center">
    <font size="4">
The businessresponsibility.ch project strengthens transparency and democratic control over the human rights performance of Swiss companies, based on the new non-financial reporting obligation.
    </font>
</td>
<tr>
</table>

## Background - Swiss Code of Obligations

By introducing a non-financial reporting obligation, Switzerland is following suit with the law that has already existed in the EU since 2014. However, there is a great lack of transparency with regard to compliance with the reporting obligation: It is unclear which companies are specifically subject to the legal obligation and it is very time-consuming to obtain an overview of which of these companies actually comply with their obligation. This lack of transparency makes it very difficult for civil society to monitor compliance with the law and to hold companies accountable.

### Indirect counter-proposal to the popular initiative

*«Für verantwortungsvolle Unternehmen – zum Schutz von Mensch und Umwelt»*

<a href="https://bit.ly/3AmYM8Q" target="blank">
  <img src="https://upload.wikimedia.org/wikipedia/commons/d/d2/Initiative_multinationales_responsables-Onex-04.jpg" height="250" /><br/>
  <p>Wikipedia ©</p> 
</a>


* [Regulation](https://www.fedlex.admin.ch/eli/oc/2021/846/de)
* [Official press release of admin.ch](https://www.bj.admin.ch/bj/de/home/aktuell/mm.msg-id-86226.html)
* [Results of the consultation process](https://www.bj.admin.ch/dam/bj/de/data/wirtschaft/gesetzgebung/verantwortungsvolle-unternehmen/ve-ber-vsotr.pdf)
* [Explanatory report](https://www.bj.admin.ch/dam/bj/de/data/wirtschaft/gesetzgebung/verantwortungsvolle-unternehmen/erlaeuterungen-vsotr.pdf)
* [FAQ from December 3, 2021 ](https://www.bj.admin.ch/dam/bj/de/data/wirtschaft/gesetzgebung/verantwortungsvolle-unternehmen/faq-internationaler-vergleich.pdf)
* [International comparison table](https://www.bj.admin.ch/dam/bj/de/data/wirtschaft/gesetzgebung/verantwortungsvolle-unternehmen/tabelle-internationaler-vergleich.pdf)

## Project Summary

With [businessresponsibility.ch](https://en.businessresponsibility.ch/), we are building a digital platform that enables citizens, activists and non-governmental organizations to check whether and how Swiss companies report on sustainability and human rights issues, as required by the new non-financial reporting obligation. The [businessresponsibility.ch](https://en.businessresponsibility.ch/) platform closes the information gap and empowers civil society to demand the information required by the law.

On [businessresponsibility.ch](https://en.businessresponsibility.ch/), interested parties can quickly and easily see which companies comply with the non-financial reporting obligation. The digital platform identifies, collects, analyzes and publishes non-financial reporting data from hundreds of Swiss companies and makes this information available free of charge.

Based on this data, [businessresponsibility.ch](https://en.businessresponsibility.ch/) enables the analysis of corporate reporting in Switzerland and thus allows civil society, authorities, legislators and last but not least the business community itself to gain a fact-based and always up-to-date insight into the development and progress of non-financial reporting in Switzerland.

<p align="center">
<img src="https://user-images.githubusercontent.com/5593131/153625662-bf233507-027a-4a49-af27-b34ec2894c77.png" height="720" />
</p>

## Architecture

The architectural layout of the businessresponsibility.ch platform can basically be represented in three clearly comprehensible and distinguishable software parts. These parts are internally connected either with system calls such as REST APIs, automated processes  or manual user interventions.

The system architecture basically consists of these three distinguishable parts:

1. Data extraction, processing and workflow management
2. Natural Language Processing (NLP) and topic classification
3. Data storage, content management (backend / APIs) and data access (frontend)

![system_diagram](https://user-images.githubusercontent.com/5593131/153616743-4fe026f9-954a-4af9-b0de-6c11b675e2a9.png)

### Design and implementation decisions

Due to time restrictions and work capacities of the project team members, we decided early on to use proprietary third-party tools to increase our chances to implement and complete a prototype on time. 

The aim of the prototype was to build a comprehensible workflow for report topic classification and highlight the potential of our solution with minimal implementation effort. With this approach, we could focus on the report topic classification and frontend development.

We currently use the [AirTable](https://www.airtable.com) as our report context database, screening and approval tool for the sustainability reports. With the [Google Search JSON API](https://developers.google.com/custom-search/v1/overview), we are able to find the concerning reports by certain search terms on the company websites. The [Integromat](https://www.integromat.com/en) is used to connect the services and automate the workflows.

![third_party_integration](https://user-images.githubusercontent.com/5593131/153617103-9fc55082-4bd6-4732-a31f-8aecf773aeb3.png)

Besides the early decision on using proprietary tools, we still laid strategic foundations to replace these tools with Open Source software components and services:

- AirTable → Strapi (Headless CMS solution)
- Integromat → Node-RED (a flow-based development tool)
- Google Search API → Node.js or Python based Crawlers (ie. Scrappy)

### 1. Data extraction, processing and workflow management

#### What do we collect and how?

![workflow](https://user-images.githubusercontent.com/5593131/153621216-2e9a86fb-165d-435c-a804-c3ea001cc7e6.png)


TBD.

#### How do we extract the data?

We store the collected data in AirTable, the extracted data in Hidora Jelastic Cloud. The text extraction service is implemented in the [report-text-extraction repository](https://github.com/bizres/report-text-extraction) using Python as the programming language. This service provides a REST API for extracting the text data from a sustainability report PDF and retrieving the context dependent information to the clients.

To start the `Extract process`, we provide a trigger via `GET` http service ‘extract’. Once the trigger is pulled the `202 Accepted` response is immediately returned to the client and the `Extract process` starts in the background on the server.

We pull all the records from the AirTable. The pulled data: ‘airtable record id’, ‘ExtractedID’, and PDF metadata, including link to PDF file. For each record we check if the ‘ExtractedID’ is provided, then we check if the corresponding PDF file is available on the local storage. If available - record is skipped.

If ‘ExtractedID’ is empty - we generate a new unique id. For not skipped records we download the corresponding PDF file and save it locally. After the download, we extract text from PDF with `pdfminer.six` library and save the TXT file locally as well. After that we prepare and save a JSON file with metadata: ‘air_id = airtable record id’, ‘id = ExtractedID’, and ‘filename = original file name’.

After extraction of every 10 records we send a bulk PATCH request back to airtable to update corresponding records with ‘ExtractedID’ or ‘ExtractedStatus’ if there is an issue with a record. 10 is the AirTable limit. The ‘ExtractedID’ field is needed to decrease/avoid issues when working on multiple servers/dev environments.

The complete extraction of 1158 records takes about 18 hours on dev PC and is about 5.23 GB.

![airtable_extraction_tool_data_flow](https://user-images.githubusercontent.com/5593131/153618491-9fa38633-6595-4fdc-b37d-d02ddaf2b52f.png)

#### What are the processes to get useful information from the extracted data?

We store three distinct pieces of information about the sustainability reports, which are the report in PDF format itself, the extracted text data and the meta-information retrieved from AirTable. These data are extracted and stored permanently in our NFS storage solution during the text extraction process and can be accessed via an API call by submitting an UUID explained in the previous chapter.

![report_classification_process](https://user-images.githubusercontent.com/5593131/153620115-47463ca2-f5f6-4b82-990a-4355bbdd2643.png)

Currently, we use the report extraction API to trigger the extraction process of the reports PDFs and subsequently saving the original PDFs and extracted data permanently in the storage solution. The data can be later retrieved either by the frontend components to display the report PDFs to the visitors or by the NLP services to execute the report classification processes. Especially the NLP services rely on the accessibility of the extracted text data and meta information.

#### Where do we store the data?

We currently utilize Hidora's Jelastic Cloud services for storing the original, extracted and processed reports data in a [Shared Storage Container](https://docs.jelastic.com/shared-storage-container/) with NFS client type for data mounting.  With this approach,  we don’t have to rely on external third-party data storage services such as Google Drive, Dropbox, AWS Storage solutions.

This storage container can be mounted in every service node in the Hidora environment and data can be accessed directly as if it were locally stored. With this approach, we can ease the data protection restrictions in internal usage.

![01-shared-storage-container-illustration](https://user-images.githubusercontent.com/5593131/153620455-25ec0795-e7d0-4989-8733-fd9a867fef63.png)

#### How do we ensure that other people access this information?

The sustainability reports must be publicly available and accessible on the Internet by the upcoming Swiss law. Accordingly, there is no urgency to make the internally stored sustainability reports accessible on our platform or via an RESTful API. 

However, on the web platform the sustainability report PDFs will be retrieved and displayed from our storage solution, in case the original report is no longer accessible on the company's original report URL. We are currently in discussion with several user groups, NGOs and governmental institutions on the accessibility of the data according to the [Open Data deployment scheme](https://5stardata.info/en/).

### 2. Natural Language Processing (NLP) and topic classification

The goal of this project is to analyze sustainability reports on their promise to report about sustainability and human rights issues as required by the new non-financial reporting obligation. To allow a structured analysis we want to classify each report in five categories. The categories are: Human Rights, Environment, Corruption, Social Concerns and Employee Concerns. For each of these categories we want to be able to tell if the report addresses related issues or not. With this information we can in a next step analyze temporal development, create lists of companies who face up to their responsibilities and those who do not, etc.

To get to this point we did not only have to collect reports and make them machine readable by extracting the text but also assess the topics which are covered by the text. All this should be done with a structured, accessible and reproducible process such that the results of our analyses are robust and explainable.

The following illustration shows schematically our process to arrive at the required results for ruther analysis. After extracting text from the reports, we tag each section of the report with one, none or multiple of the considered categories. If a section is tagged with a category then this means that the section somehow assesses a topic relevant to the category. If a report has enough sections assessing one of the categories then the whole report is considered to report on the topic.

![nlp_process](https://user-images.githubusercontent.com/5593131/153622339-c1602b63-5317-41e4-aefe-5844b63a6dc7.png)

Classifying the reports is a non-trivial task. On one hand the sheer amount of text is too much for a human classifier. On the other hand a manual approach of classifying the reports is neither accessible nor reproducible since different human beings assess text in different ways and afterwards it is not clear why a text was e.g. labeled as addressing social concerns or why not. Our answer to solve this problem was the development of an algorithm which automatically classifies texts. This algorithm can not only do a day’s work of a human in a few seconds but also (and more importantly) be validated and rerun whenever necessary. The validation of the algorithm allows us to assess how stringent and accurate our classifications are and also to show the design and parameters of our model which explains why sections are tagged in a transparent and reproducible way. In the machine learning community this way of processing text is often called Natural Language Processing or NLP for short.

The following illustration shows the process of training our model. First a sample of reports is tagged by humans. Those human taggers have professional expertise with the relevant topics. Furthermore their classifications are compared and only sections are used where consensus exists.

![nlp_tagging_sections](https://user-images.githubusercontent.com/5593131/153622898-2076d29f-3eee-4a53-aa4e-bbab3a82bdc5.png)

#### Parameters of our Model

Our model has three high level parameters. First we had to decide how we convert text passages into numerical values which then can be processed by our model. This step is called **text vectorization** since at the end the text is represented by a vector (e.g. a list) of numerical values. Secondly we had to decide which **prediction algorithm** to use to create the labels. Finally we had to decide for which **performance metric** we want our algorithm to be optimized.

#### Text Vectorization

To represent the text numerically, we tried two approaches. The first approach is called [bag-of-words](https://en.wikipedia.org/wiki/Bag-of-words_model) and was completely generated by us based on the training sample. The second approach was taking the [vectorization model of Spacy](https://spacy.io/usage/models) to generate our own vectors. Those models are very complex and already pre trained by the Spacy community. Therefore they are more accurate but also less transparent.

#### Prediction algorithm

As prediction algorithms we tried [logistic regression](https://en.wikipedia.org/wiki/Logistic_regression) and [naive bayes](https://en.wikipedia.org/wiki/Naive_Bayes_classifier) classifiers. Both algorithms are well known and often used in the machine learning community.

#### Performance Metric

When classifying texts there are two two errors which can happen. False Positives label a text as belonging to a category which it in truth does not. False Negatives do not label a text as belonging to a category even though in truth it should be. We tried two approaches where we first optimized our model for **accuracy**. This means that we try to minimize false positives and false negatives overall but do not care if there are more or less False Negatives than False Positives. Our second approach was optimizing the model for **recall** which means that we tried to minimize the False Negatives. In other words by optimizing for recall we tried to really find all texts belonging to a category even if we risk wrongfully labeling too many of the texts as belonging to this category.

## Technologies

Following technologies are currently selected:
* Third-Party tools: AirTable, Integromat, Google Search API - used for Rapid Prototyping purposes
* Jelastic Cloud environment: Hidora.io (https://hidora.io)
* Loadbalancer: Nginx
* Backend: Node.js with Strapi (https://strapi.io)
* Frontend: Node.js with Next.js / React (https://nextjs.org/)
* Database: PostgreSQL
* Machine Learning and NLP: Python, fastText
* Development and Production Environment with Docker

## Project roadmap

## Contributing to the project

We aim to be a community driven project, which embraces the broad range of opinions, people with cultures and different mind-sets, and technological backgrounds. We are sure that a close collaboration with the Open Source communities and Institutions is essential for the success of this project. Therefore, we encourage and welcome everybody, who wants to participate and make a contribution to this project.

You can reach us either on our website [businessresponsibility.ch](https://en.businessresponsibility.ch/) or [GitHub project page](https://github.com/bizres) or with our official E-Mail address [info@businessresponsibility.ch](mailto:info@businessresponsibility.ch).


## Team

We are an interdisciplinary team of political scientists, economists, environmental activists and software developers that support the [businessresponsibility.ch](https://en.businessresponsibility.ch/) project. This is a non-profit startup project that aims to strengthen transparency and democratic oversight of the human rights performance of Swiss companies.

We are building a digital platform to monitor company performance vis a vis non-financial reporting obligations. We are currently funded by the [Prototype Fund](https://prototypefund.opendata.ch/), which supports innovative open source projects that strengthen democratic participation in Switzerland through digital solutions.


* David Weiss (Project Lead)
* Dina Pomeranz (Strategic Advisor)
* Deborah Kistler (Project Manager)
* Kobbina Awuah (Business Manager)
* Cahit Atilgan (Tech Lead)
* Nickolay Golomysov (Tech Developer)
* Johannes Hool (Tech Developer)
* Miguel Vazquez Vazquez (Tech Developer)
* Schahin Bajka (Report Analysis Support)

## Feedback
We welcome any feedback concerning our project! Please send us a message to our official E-Mail address [info@businessresponsibility](mailto:info@businessresponsibility) or reach out for specific questions our team members:

* Project specific questions and feedback: David Weiss and Dina Pomeranz
* Technology specific questions: 
** System architecture / GitHub / Development: Cahit Atilgan
** Machine Learning / Natural Language Processing: Nickolay Golomysov or Johannes Hool

## License
This project is currently using the [3-Clause BSD license](https://opensource.org/licenses/BSD-3-Clause), which allows almost unlimited freedom with the software as long as the BSD copyright and license notice are included

## Acknowledgments
