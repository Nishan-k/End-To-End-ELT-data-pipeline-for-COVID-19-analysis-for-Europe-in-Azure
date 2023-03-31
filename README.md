# COVID project implemented using Azure Service End-to-End ELT pipeline

<img src="./images/workflow.png" alt="DE-workflow" title="Data Pipeline Worflow"><br>

## Goal Of The Project:
The data used for the project is of Europe. The data has been extracted from EuroStat and European Centre for Disease Prevention and Control (ECDC) available in GitHub.
<ul>
    <li>The main objective of this project was to implement the Azure services like  <b> Azure Data Factory, Azure Blob Storage, Azure Data Lake to extract, transform and load the data into Azure SQL database and finally visuazlize the data using Tableau. </b></li>
</ul>

## Meta data about the files used in the project:
<table>
<thead>
  <tr>
    <th>File Names</th>
    <th>Description of file</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>1. population_by_age.tsv.gz (EuroStat/Blob)</td>
    <td>This is a zipped file that contains the population data of countries.</td>
  </tr>
  <tr>
    <td>2. cases_deaths.csv (ECDC from GitHub)</td>
    <td>This csv contains the number emerging Covid Cases and Deaths followed by the each day.</td>
  </tr>
  <tr>
    <td>3. hospital_admissions.csv (ECDC from GitHub)</td>
    <td>This csv file contains the Daily Hospital Admissions, Daily ICU admissions, Weekly Hospital Admissions per 100k, Weekly ICU Admissions per 100k.</td>
  </tr>
  <tr>
    <td>4. testing.csv (ECDC from GitHub)</td>
    <td>This csv file contains the emerging covid cases, tests being done, testing_rate and covid postive_rate on the weekly basis.</td>
  </tr>
  <tr>
    <td>5. Country response (ECDC from GitHub)</td>
    <td>How different countries responded to the alarming cases and impacts to COVID outbreak.</td>
  </tr>
  <tr>
    <td>6. lookups (Blob)</td>
    <td>Other miscellaneous files like Calendar Lookup/dim and Country lookup/dim files used.</td>
  </tr>
</tbody>
</table>

## Table of Contents:
<ol>
    <li>Data Ingestion</li>
    <ol type='a'>
        <li><a href="#populationdata"> <b>Ingest Population data from Azure Blob Storage to Azure Data Lake </a></b></li>
    </li>
    <li><a href="#ecdcdata"> <b>Ingest ECDC data from GitHub to Azure Data Lake using HTTP linkedservice</a></b></li>
    </li>
    </ol>
    <li>Data Transformation</li>
    <ol type='a'>
    <li><a href="#dataflow"> <b>Transformation using Data Flow</b> </a></li>
    </ol>
</ol>
<h2>1. Data Ingestion </h2>
<h3 id ="populationdata">a. Ingest Population data from Azure Blob Storage to Azure Data Lake:</h3>
<p>First, we are ingesting the data from Azure Blob Storage to the raw folder in Azure Data Lake.</p>

<br>
<br>
<b>Pipeline to ingest data from Azure Blob Storage for population data:</b>
<br>
<br>
<img src="./images/1.ingest population data.png" alt="DE-workflow" title="Data Pipeline Worflow"><br>

<b>STEPS involved: </b>
<ol>
    <li>Validation:</li>
    <p>The first phase is the data validation, if the file exists in the Azure blob storage then we are good to go and will proceed to the next stage.<p>
    <li>Get Metadata:</li>
    <p>Once, the availability of the data has been verified, we will extract the metadata, here we are extracting the number of columns of the data being processed.</p>
    <li>If Condition:</li>
    <p>Here, we will check the number of columns, if the number of columns is greated than 13, then it is an error, the number of columns we are expecting for our dataset to have is 13. So, if the dataset has more than 13 columns, we will send an E-mail notifying us that the data has more number of columns. Else, if the condition matches, we will copy the data to the DataLake Gen-2 in the raw/population folder and then delete it once it has been copied to clean up the space.</p>
</ol>
<br>
<h3 id ="ecdcdata">b. Ingest ECDC data from GitHub to Azure Data Lake using HTTP linked-service:</h3>
<p>In total we will be ingesting 4 files:
    <ul>
        <li> COVID-19 new cases and deaths by country
        <li> COVID-19 Hospital admissions & ICU cases
        <li> COVID-19 Testing numbers
        <li> Country Response to COVID-19
    </ul>
</p>
<b>Pipeline to ingest data from Azure Blob Storage for population data:</b>
<br>
<br>
<img src="./images/2.ingest ecdc data.png" alt="DE-workflow" title="Data Pipeline Worflow"><br>
<b>STEPS involved: </b>
<ol>
    <li>Lookup:</li>
    This operation will lookup for a JSON file created as a dataset. This JSON file contains the information such as:
    <br>
    - BaseURL (linkedservice)<br>
    - RelativeURL (dataset)<br>
    - FileName (file to be saved as)
    <li>ForEach:</li>
    Once the file has been found in the Lookup phase, we will loop through the JSON file and pass each record to the copy activity.
    <li>Copy Activity:</li>
    In copy activity, we will pass the <b> BaseURL, RelativeURL and the Filename</b> to pull the data from GitHub repo and copy it to the DataLake Gen-2 in the raw folder.
</ol>

For this pipeline to be executed, I have created a scheduled trigger that would execute the pipeline to ingest the ECDC data every hour without an end date.

<img src="./images/3.ecdc trigger .png" alt="trigger"><br>

<h2> 2. Data Transformation </h2>
<h3 id="dataflow"> a. Transformation using Data flow </h3>
<p><b>1. Transforming cases and deaths data using Data Flow in Azure Data Factory.</b></p>

<b>Data flow for cases and deaths transformation: </b>
<img src="./images/2.dataflow for casesanddeaths.png" alt="dataflow"><br>

Here, the dataset goes through multiple transformation phases, and finally the processed data is stored in Data Lake/processed folder.

<p>2. Transforming Hospital Admissions data using Data flow </p>
<img src="./images/3.dataflow for hospital data transformation.png" alt="dataflow"><br>

<p>3. Transforming Population data using Data Bricks </p>

Here, I have mounted Databricks with Azure Data lake, and then transformed the Population data and then stored back to the Data Lake in Processed folder.

<p>4. Copying the final clean data to the Azure SQL database </p>
In total, I have created 3 tables: <br>
- cases and deaths <br>
- hospital admissions daily <br>
- testing (COVID testing) 

<p>5. Visualize the data </p>
Connecting to Azure SQL database from Tableau:
<img src="./images/connection.png" alt="dataflow"><br>

<h3>Dashboard Visualizing Cases and Death Counts Per Country </h3>
<div class='tableauPlaceholder' id='viz1680297334822' style='position: relative'><noscript><a href='#'><img alt='Covid Case and Death Analysis ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;P8&#47;P8XS9DHNH&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='path' value='shared&#47;P8XS9DHNH' /> <param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;P8&#47;P8XS9DHNH&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /><param name='filter' value='publish=yes' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1680297334822');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1400px';vizElement.style.height='927px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1400px';vizElement.style.height='927px';} else { vizElement.style.width='100%';vizElement.style.height='977px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

