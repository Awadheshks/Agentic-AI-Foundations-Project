# Agentic-AI-Foundations-Project

#### To build a product recommendation chatbot with Amazon Bedrock Agents 
This project will demonstrate how to build an e‑commerce chatbot using Amazon Bedrock Agents, configuring your existing APIs so the bot can invoke them and complete user tasks directly from the conversation

### Architecture
The project follows a serverless architecture of Amazon Bedrock calling AWS Lambda functions which do some actions on a set of Amazon DynamoDB tables, and utilizing Knowledge Bases for Amazon Bedrock to incorporate data from files in Amazon Simple Storage Service (Amazon S3).

We will explore below scenarios 
### Explore Products
### Cart Actions
### Integration with Amazon Personalize
### Adding Gift Wrapping Knowledge base
### Multi-Agent Collaboration


![img]()


Let's follow the steps :

## Explore Products
### 1. Create an Agent
 Create an Agent from Amazon Bedrock Console > Agents , you can use _product-recommendation-agent_ as the name:

![img]()

### 2. Select Model
As Anthropic Claude Sonnet 4.5 as the model, deselect the 'Bedrock Agent Optimized' checkbox to see it

![img]()

1. Select Use an existing service role as the Agent resource role, then select the role starting with producttableandapi-ws-IAMRole00AmazonBedrock.
2. Use the following as the instructions for the agent:
   
   _"you are a product recommendations agent for gift products, the user is trying to buy a gift for someone and you are trying to help identify the best products based on the    filters in the action groups, ask questions to identify at least one of the input filters, gender, category or occasion.
do not recommend any products that are not retrieved from the products API.
do not ask about the gender if it is obvious from the user input already.
Always start by getting the full list of products from the API so you can know the proper filter values to be used in the API parameters.
always use a single value for each filter field, and adhere to the filtration values based on the first API call.
And never tell the user about the API and its details."_

3. Enable User input in Additional settings:
   
### 3. Add Action Group
1. Navigate to the Action Groups section, then choose “Add” to add an Action Group to enable the Agent to invoke the Lambda Function, put the name as get-product-recommendations and choose Action Group Type “Define with API Schemas”:
2. Select the ‘GetProductDetailsFunction’ Lambda function for Action Group Invocation:
3. Choose ‘Define Via inline schema editor’ for Action Group Schema:And Use the following OpenAPI schema in the in-line OpenAPI schema section:
4. 
_   _"{
  "openapi": "3.0.0",
  "info": {
    "title": "Product Details API",
    "version": "1.0.0",
    "description": "This API retrieves product information. Filtering parameters are passed as query strings. If query strings are empty, it performs a full scan and retrieves the full product list."
  },
  "paths": {
    "/products": {
      "get": {
        "summary": "Retrieve product details",
        "description": "Retrieves a list of products based on the provided query string parameters. If no parameters are provided, it returns the full list of products.",
        "operationId":"getProducts",_
        "parameters": [
       _   {
            "name": "product_name",
            "in": "query",
            "description": "Retrieve details for a specific product",
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "category",
            "in": "query",
            "description": "Filter products by category",
            "schema": {
              "type": "string"
            }
          },
          {_
            "name": "gender",
            "in": "query",
            "description": "Filter products by gender",
            "schema": {
              "type": "string"
          _  }
          },
          {
            "name": "occasion",
            "in": "query",
            "description": "Filter products by occasion",
            "schema": {
              "type": "string"
            }
          }
        ],_
      _  _"responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "product_name": {
                        "type": "string"
                      },
                      "category": {
                        "type": "string"
                      },
                      "gender": {
                        "type": "string"
                      },
                      "occasion": {
                        "type": "string"
                      },
                      "product_description":_ {
                   _     "type": "string"_
                       }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }

}"

4. Choose “Create” to create the action group, then on the Agent builder page, Choose “Save” and then “Prepare” on the top of the page. then you can use the ‘Test Agent’ section on the right to start testing having conversations with the chatbot.

![img]()

5. Check Sample Conversation and Responses through the rationale for each response by choosing Show trace >, see how the Agent decides to use different API filters based on the discussion. Go to Orchestration and Knowledge base

![img]()

Lets proceed to next sections to give our chatbot more capabilities

## Cart Actions

### 1. Edit the Agent
![img]()

### 2. Add new Agent Group
![img]()

### 3. Schema for get cart
![img]()

### 4. Schema for add item to cart
![img]()


## Integration with Amazon Personalize
### 1. Edit the Agent
![img]()

### 2. Add new Action groups
![img]()


## Adding Gift Wrapping Knowledge base
### 1. Synchronize the Knowledge Base data source
![img]()

### 2. Edit the Agent
![img]()

### 3. Add new Knowledge Base to Agent
![img]()


## Multi-Agent Collaboration
### 1. Create a Product Recommendation Agent
   ![img]()
   
### 2. Create the Cart Management agent
   ![img]()
   
### 3. Create the Supervisor Agent
   ![img]()
   
### 4. Configure Multi-Agent Collaboration
   ![img]()
   
