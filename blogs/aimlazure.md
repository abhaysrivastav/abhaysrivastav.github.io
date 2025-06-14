# Azure AI/ML Infrastructure

One of the most essential component of Azure's AI ML Infrastructure is the **Azure Machine Learning Service**. This service act act as a backbone of any machine learning project. If offers an integrated enviornment where you can train models, experiment with different algorithms and manage entire machine learning workflow.

**Azure Databricks** provides a powerful, collaborative workspace optimized for big data analytics and Machine Learning. It is built on Apache Spark, making it incredibally efficient for processing large dataset.

Azure provides several storage options including **Azure Datalake Storage** , **Azure SQLDatabase** and **Azure Blob Storage**. 

Once the models are trained and ready for deployment, Azure provides a range of services to get them into the production. We can use **Azure Kubernetes Service** if we need scaling. If we need straight forward deployment we can use **Azure Machine Learning Service**. 

For Monitoring, Azure provides services like **Azure Monitor** and **Application Insights**. 

## Data Sources and pipelines, frameworks and plateforms

A data pipeline is a series of process that automate the flow of data from one point to another, transforming it along the way to prepare it for the analysis or for machine learning. Key components of data pipeline includes **data sources**, **data ingestion tools**, **data transformation processes** and **data storage**. 

In Azure, the most common frameworks include Tensorflow, Pytorch, Scikit-learn and azure machine learning SDK.

Azure offers several deployment options depending on your needs, including **Azure Kubernetes Service**, or AKS, **Azure Machine Learning Service**, and **Azure App Services**. Azure Kubernetes Service is ideal for <ins>deploying models at scale</ins>. It allows you to containerize your models and deploy it across a cluster of virtual machines, providing high availability as well as scalability. This is particularly useful for applications that require real-time predictions or need to handle a large volume of requests simultaneously. **Azure Machine Learning Service** simplifies the deployment process by allowing you to <ins>deploy models as web services</ins> with just a few clicks.**Azure App Services** is another option for deploying models as <ins>part of a web application</ins>, offering a managed environment with built-in scaling and monitoring. 

## Selecting the right model deployment strategy in Microsoft Azure

### Deployment speed

**Azure Machine Learning service**: This service offers a streamlined way to deploy models with minimal setup time. It supports deploying models as RESTful web services, allowing for rapid deployment and easy integration into existing applications.

**Azure Kubernetes Service (AKS)**: If we require high availability and rapid scaling, AKS can quickly deploy containerized models. However, it requires more initial setup and familiarity with Kubernetes.

### Cost efficiency


**Azure Functions**: For infrequent or lightweight deployments, Azure Functions offers a serverless computing option where you only pay for the execution time of your function. This can be cost-effective for models that don't require constant availability.

**Azure Container Instances (ACI)**: ACI is a lower-cost option for deploying containerized models without the need for orchestration. It’s ideal for small-scale or temporary deployments.

**Reserved instances**: For long-term deployments, consider using reserved instances, which offer significant discounts compared to pay-as-you-go pricing.

### Ease of use

**Azure Machine Learning Studio:** This low-code/no-code environment allows for easy deployment with a graphical interface. It’s ideal for teams that may not have deep DevOps or cloud computing expertise.

**Azure App Service:** This option offers a straightforward way to deploy web applications and APIs. If your model needs to be part of a web-based application, Azure App Service provides an easy-to-manage environment with integrated deployment pipelines.

### Scalability

**Azure Kubernetes Service (AKS)**: For large-scale, enterprise-level deployments, AKS provides robust scalability features. It supports autoscaling, load balancing, and orchestrating multiple container instances.

**Azure Batch**: If your deployment involves processing large volumes of data or requires parallel execution of multiple models, Azure Batch offers a scalable solution that can distribute workloads across many virtual machines.

### Updates and maintenance

**Azure DevOps**: Integrating your deployment pipeline with Azure DevOps allows for continuous integration and continuous deployment (CI/CD). This makes it easier to push updates, roll back changes, and automate testing.

**Azure monitoring tools**: Azure provides a range of monitoring tools such as Azure Monitor, Log Analytics, and Application Insights. These tools help you track model performance, detect anomalies, and troubleshoot issues in real time.

## Data Sources & pipelines
A data pipeline is a set of automated processes that take raw data from your sources, clean and transform it, and then ultimately deliver it to a destination.**Azure Data Factory** is cloud based data-integration service that allows to create, schedule and orchestrate data pipelines.It supports wide range of data sources and can handle both structured and unstructured data.  For more complex transformation **Azure Databricks** provides a collaborative platform that is optimized for big data analytics and machine learning applications.


### Key stages of data pipeline


1) **Data Ingestion**

**Role**: This is the first stage of the pipeline, where raw data is collected from various sources. Data ingestion can happen in real time (streaming data) or in batches (bulk data transfers). The method you choose depends on the nature of your data and the requirements of your project.

**Tools**: In Azure, data ingestion is often managed by **Azure Data Factory**, which can connect to a wide range of data sources, including on-premises databases, cloud storage, and APIs. It allows you to schedule and automate data transfers, ensuring that your pipeline stays up to date.

2) **Data Processing/ Transformation**

**Role**: Once data is ingested, it must be processed and transformed to make it suitable for analysis or model training. This can involve cleaning (removing duplicates, filling in missing values), normalizing (scaling numerical values), or transforming (aggregating data, joining tables) the data.

**Tools**: **Azure Databricks** is a powerful tool for this stage, offering a collaborative environment built on Apache Spark. It allows for large-scale data processing and complex transformations, making it ideal for preparing data for machine learning.

3) **Data Storage**:

**Role**: After processing, the data is stored in a location where it can be easily accessed by data analysts, data scientists, and AI/ML models. The choice of storage depends on the type of data and how it will be used.

**Tools**: Azure offers several storage solutions: 

**Azure Data Lake Storage** is optimized for storing vast amounts of raw, unstructured data.

**Azure SQL Database** is ideal for storing structured data that requires advanced querying capabilities.

**Azure Blob Storage** is a versatile storage solution for unstructured data like text files, images, and videos.

4) **Data Access and utilization**

**Role**: The final stage involves making processed and stored data available for analysis or input into machine learning models. This is where data scientists and engineers interact with the data, using it to train models, generate reports, or drive decision-making processes.

**Tools**: Data can be accessed through various services, such as **Azure Machine Learning Service** for model training or Power BI for business analytics.


## Data Acquisition Method
* Direct Data Collection
* Third Party Data Soruces
* Web Scraping
* APIs 
* Publicly available datasets

## Step-by-step guide to configuring resources for AI/ML projects

## Compute instances

Compute instances in Azure are virtual machines specifically optimized for data science and machine learning tasks. They allow you to execute training jobs, run Jupyter notebooks, and develop models in an interactive environment. These compute instances are highly versatile and can be tailored to meet the unique needs of different AI/ML projects.

### Types of compute

**1) CPU-based instance**

**2) GPU-based instance** 

### Configuration Steps:

1) Access Azure ML workspace

2) Select the compute tab

3) Create a new compute instance

4) Choose the VM size

5) Configure the scaling option

6) Manage Access and Permission

## Datastore 

Datastores in Azure provide a simple and secure way to connect storage accounts to your machine learning workspace. Datastores act as a bridge between your ML workspace and the underlying data storage, making it easier to manage, maintain, and access the data required for AI/ML experiments. 

### Storage Options

1) Azure Blob Storage 
2) Azure Data Lake
3) Azure File Storage
4) Azure SQL Database 
5) Azure Cosmos DB

[Storage Options](<resources/azureaiml/Explanation of storage solutions .pdf>)

[Data Storage Solution](<resources/azureaiml/Implementing data storage solutions .pdf>)

### Configuration steps

1) Access the datastore section
2) Register a new datastore 
3) Provide the authentication details
4) Set access permission

## Data ingestion pipelines

The remaining of this reading will guide you through the following steps:

Step 1: Identify data sources (Database, APIs, Flat files, logs or other types of structured & unstrctured data)

Step 2: Choose an ingestion method (Batch ingestion, Streaming ingestion)

Step 3: Extract data (API connectors, database queries, file readers)

Step 4: Data transformation and quality checks (Cleaning, Deduplication, Enrichment, Restructuring)

Step 5: Load data into storage (Either into data warehouse or datalake)

Step 6: Schedule and monitor 

## Data preprocessing 

### Common data pre-processing techniques:

1) Data Cleaning 
2) Missing data handling 
3) Feature Scaling 
4) Data encoding 
5) Feature selection and dimensionality reduction
6) Outlier detection and handling
7) Data transformation
8) Data binning 
9) Sythetic data generation 
10) Data shuffling and splitting 

[data pre-processing Resource](<resources/azureaiml/Explanation of preprocessing techniques.pdf>)






