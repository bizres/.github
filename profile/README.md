# Welcome to the "businessresponsibility.ch" project! 👋

<table border="0" style="display:contents">
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

![businessresponsibility.ch](/bizres_screenshot.png)

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

### Data extraction, processing and workflow management

We store the collected data in AirTable, the extracted data in Hidora Jelastic Cloud. The text extraction service is implemented in the [report-text-extraction repository](https://github.com/bizres/report-text-extraction) using Python as the programming language. This service provides a REST API for extracting the text data from a sustainability report PDF and retrieving the context dependent information to the clients.

To start the `Extract process`, we provide a trigger via `GET` http service ‘extract’. Once the trigger is pulled the `202 Accepted` response is immediately returned to the client and the `Extract process` starts in the background on the server.

We pull all the records from the AirTable. The pulled data: ‘airtable record id’, ‘ExtractedID’, and PDF metadata, including link to PDF file. For each record we check if the ‘ExtractedID’ is provided, then we check if the corresponding PDF file is available on the local storage. If available - record is skipped.

If ‘ExtractedID’ is empty - we generate a new unique id. For not skipped records we download the corresponding PDF file and save it locally. After the download, we extract text from PDF with `pdfminer.six` library and save the TXT file locally as well. After that we prepare and save a JSON file with metadata: ‘air_id = airtable record id’, ‘id = ExtractedID’, and ‘filename = original file name’.

After extraction of every 10 records we send a bulk PATCH request back to airtable to update corresponding records with ‘ExtractedID’ or ‘ExtractedStatus’ if there is an issue with a record. 10 is the AirTable limit. The ‘ExtractedID’ field is needed to decrease/avoid issues when working on multiple servers/dev environments.

The complete extraction of 1158 records takes about 18 hours on dev PC and is about 5.23 GB.

![airtable_extraction_tool_data_flow](https://user-images.githubusercontent.com/5593131/153618491-9fa38633-6595-4fdc-b37d-d02ddaf2b52f.png)


## Team

We are an interdisciplinary team of political scientists, economists, environmental activists and software developers that support the [businessresponsibility.ch](https://en.businessresponsibility.ch/) project. This is a non-profit startup project that aims to strengthen transparency and democratic oversight of the human rights performance of Swiss companies.

We are building a digital platform to monitor company performance vis a vis non-financial reporting obligations. We are currently funded by the Prototype Fund, which supports innovative open source projects that strengthen democratic participation in Switzerland through digital solutions.



* David Weiss (Project Lead)
* Dina Pomeranz (Strategic Advisor)
* Deborah Kistler (Project Manager)
* Kobbina Awuah (Business Manager)
* Cahit Atilgan (Tech Lead)
* Nickolay Golomysov (Tech Developer)
* Johannes Hool (Tech Developer)
* Miguel Vazquez Vazquez (Tech Developer)
