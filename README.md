# Agentic-AI-Foundations-Project

#### To build a product recommendation chatbot with Amazon Bedrock Agents 
This project will demonstrate how to build an e‑commerce chatbot using Amazon Bedrock Agents, configuring your existing APIs so the bot can invoke them and complete user tasks directly from the conversation

### Architecture
The project follows a serverless architecture of Amazon Bedrock calling AWS Lambda functions which do some actions on a set of Amazon DynamoDB tables, and utilizing Knowledge Bases for Amazon Bedrock to incorporate data from files in Amazon Simple Storage Service (Amazon S3).

![img]()


### Senarios
 * Explore Products
 * Cart Actions
 * Integration with Amazon Personalize
 * Adding Gift Wrapping Knowledge base
 * Multi-Agent Collaboration


### Prerequisites:

If using your own AWS account, follow these steps to create a new Admin user for running this project.

1. Select AWS Access Type
 (i) Enable AWS Management Console Access
 (ii) Set Console Password as  "Custom assword"
 (iii) Uncheck-"Reset password Reset"
2. Attach the AdministratorAccess IAM Policy as "AdministratorAccess"
3. Click to create the new user.
4. Take note of the login URL and save it.
5. Login as the administrator user via URL above.
6. Install workshop content - The workshop content is prepared as CloudFormation template, you can deploy it easily by following instructions below
7. Click "Launch in the AWS Console" button to initiate installation of the required resources in CloudFormation.
8. Choose Next in the bottom right of the page. In Specify stack details page, enter producttableandapi-ws for Stack name. Let everything else as default setting, and then choose Next.
9. Choose Next in Configure stack options page.

10. In the last page, select all checkboxes in Capabilities and transforms pane, and then choose Submit. Wait until stack status is CREATE_COMPLETE. It can take around 5-10 minutes for the stack creation to finish.

* You will be charged for the deployed resources, so make sure you clean up after you are done with the workshop.


Let's Start the workshop by following the steps :

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
In this section, you’ll add two key functions to the agent: adding items to a shopping cart and retrieving cart details. The chatbot will invoke prebuilt Lambda functions—_GetCartFunction_ and _AddToCartFunction_ to manage the _producttableandapi-ws-Cart-XXXXX_ DynamoDB table (with user_id and product_name attributes), created during workshop setup.

### 1. Edit the Agent
(i) Choose the Agent you created in the previous section of the workshop, and choose Edit in Agent Builder
![img]()

(ii) Use the following as the updated instructions for the agent:

_you are a product recommendations agent for gift products, the user is trying to buy a gift for someone and you are trying to help identify the best products based on the filters in the action groups, ask questions to identify at least one of the input filters, gender, category or occasion.
do not recommend any products that are not retrieved from the products API.
do not ask about the gender if it is obvious from the user input already.
Always start by getting the full list of products from the API so you can know the proper filter values to be used in the API parameters.
always use a single value for each filter field, and adhere to the filtration values based on the first API call.
And never tell the user about the API and its details.
After recommending products ask the user if they want to add any products to the cart, then use the add to cart API to add it, ask the user about his email and use it as user id and use it along the whole conversation and in any cart API calls, reply to the user with the user id after first cart addition to be used in later additions.
after adding an item to the cart, retrieve the cart items from the get cart API and display it to the user, the user can ask about the items in the cart at any time, use the get cart api to answer that._

### 2. Add new Agent Group
(i) Navigate to the Action Groups section, then choose “Add” to add an Action Group to enable the Agent to invoke the GetCartFunction Lambda Function, put the name as get-cart and choose Action Group Type “Define with API Schemas”

(ii) Select the ‘GetCartFunction’ Lambda function for Action Group Invocation and Select ‘Define Via inline schema editor’ for Action Group Schema.

![img]()

(iii) Schema for get cart
_{
  "openapi": "3.0.0",
  "info": {
    "title": "Get Cart API",
    "version": "1.0.0",
    "description": "This API retrieves the items in a user's cart."
  },
  "paths": {
    "/cart": {
      "get": {
        "summary": "Get the user's cart",
        "description": "Retrieves the list of products in the user's cart.",
        "operationId":"getCart",
        "parameters": [
          {
            "name": "userId",
            "in": "query",
            "description": "The ID of the user",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
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
                      }
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad Request",
            "content": {
              "application/json": {
                "schema": {
                  "type": "string"
                }
              }
            }
          }
        }
      }
    }
  }
}_

![img]()

(iv) Choose “Create” to create the action group.

### 4. Add new Agent Group
(i) Add another Action Group in the Action Groups section, Choose “Add” to add an Action Group to enable the Agent to invoke the AddToCartFunction Lambda Function, put the name as add-item-to-cart and choose Action Group Type “Define with API Schemas

(ii) Select the ‘AddToCartFunction’ Lambda function for Action Group Invocation and Select ‘Define Via inline schema editor’ for Action Group Schema:

![img]()

(iii) Schema for add item to cart
_{
  "openapi": "3.0.0",
  "info": {
    "title": "Add to Cart API",
    "version": "1.0.0",
    "description": "This API adds a product to a user's cart."
  },
  "paths": {
    "/cart": {
      "post": {
        "summary": "Add a product to the cart",
        "description": "Adds a product to the user's cart. If the product already exists in the cart, it returns an error.",
        "operationId":"postCart",
        "parameters": [
          {
            "name": "userId",
            "in": "query",
            "description": "The ID of the user",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "productName",
            "in": "query",
            "description": "The name of the product to add to the cart",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "string"
                }
              }
            }
          },
          "400": {
            "description": "Bad Request",
            "content": {
              "application/json": {
                "schema": {
                  "type": "string"
                }
              }
            }
          }
        }
      }
    }
  }
}_

(iv) Choose “Create” to create the action group.

(v) Then on the Agent builder page, Choose “Save” and then “Prepare” on the top of the page. Now you can use the ‘Test Agent’ section on the right to start testing having conversations with the chatbot.

(vi) Test the Agent.

![img]()
![img]()
![img]()

- Now lets proceed to next section to give our chatbot even more capabilities

## Integration with Amazon Personalize
In this section, we’ll add a simulated integration with Amazon Personalize, a fully managed ML service that generates personalized item recommendations without requiring ML expertise. The goal is to use Amazon Personalize to increase average cart size by suggesting relevant and trending items that boost order value and reduce cart abandonment. The chatbot will call a preconfigured Lambda function, _GetPersonalizeRecommendationFunction_, which simulates “Customers who bought X also bought Y” by recommending products frequently purchased alongside the item the agent just added to the cart.

### 1. Edit the Agent

(i) Choose the Agent you created in the previous section of the workshop, and choose Edit in Agent Builder

![img]()

(ii) Use the following as the updated instructions for the agent.

_you are a product recommendations agent for gift products, the user is trying to buy a gift for someone and you are trying to help identify the best products based on the filters in the action groups, ask questions to identify at least one of the input filters, gender, category or occasion.
do not recommend any products that are not retrieved from the products API.
do not ask about the gender if it is obvious from the user input already.
Always start by getting the full list of products from the API so you can know the proper filter values to be used in the API parameters.
always use a single value for each filter field, and adhere to the filtration values based on the first API call.
And never tell the user about the API and its details.
After recommending products ask the user if they want to add any products to the cart, then use the add to cart API to add it, ask the user about his email and use it as user id and use it along the whole conversation and in any cart API calls, reply to the user with the user id after first cart addition to be used in later additions.
before item addition to the cart, use the get personalize recommendation API to get products other customer has bought based on the product that you just added to the cart, recommend that to the user by telling them that other customers also bought this along with the product that you just added and ask if they want to also add it to the cart as well.
after adding an item to the cart, retrieve the cart items from the get cart API and display it to the user, the user can ask about the items in the cart at any time, use the get cart api to answer that._

### 2. Add new Action groups
(i)Navigate to the Action Groups section, then choose “Add” to add an Action Group to enable the Agent to invoke the GetPersonalizeRecommendationFunction Lambda Function, put the name as get-amazon-personalize-recommendation and choose Action Group Type “Define with API Schemas”.
(ii) Select the ‘GetPersonalizeRecommendationFunction’ Lambda function for Action Group Invocation and Select ‘Define Via inline schema editor’ for Action Group Schema:
(iii) And Use the following OpenAPI schema in the in-line OpenAPI schema section:

__{
  "openapi": "3.0.0",
  "info": {
    "title": "Get Product Recommendation API",
    "version": "1.0.0",
    "description": "This API retrieves a recommended product based on the input product name."
  },
  "paths": {
    "/personalize-recommendation": {
      "get": {
        "summary": "Get a recommended product from Amazon personalize",
        "description": "Retrieves a recommended product based on the input product name using Amazon Personalize.",
        "operationId":"getPersonalize",
        "parameters": [
          {
            "name": "product_name",
            "in": "query",
            "required": true,
            "description": "The name of the product for which to get a recommendation",
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
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
                      "product_description": {
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
                      }
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad Request",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "message": {
                      "type": "string"
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
}_

(iv) Choose “Create” to create the action group.

(v) Then on the Agent builder page, Choose “Save” and then “Prepare” on the top of the page. Now you can use the ‘Test Agent’ section on the right to start testing having conversations with the chatbot.

(vi) Test the Agent.

![img]()
![img]()
![img]()

- Now lets use another source of data in our Agent, in the next section we will use Amazon Bedrock Knowledge bases to suggest gift wrapping ideas

## Adding Gift Wrapping Knowledge base
In this section, we’ll extend the agent with a new capability: generating gift‑wrapping ideas from a knowledge base using Knowledge Bases for Amazon Bedrock. By using Knowledge Bases, we can provide foundation models and agents with contextual information from your organization’s private data sources for RAG, enabling more relevant, accurate, and tailored responses.

### 1. Synchronize the Knowledge Base data source

(i) Go to the Knowledge Bases for Amazon Bedrock  console.

(ii) Choose the 'GiftWrappingKnowledgeBase'

(iii) In the Data source section, select the 'GiftWrappingDataSource' and choose Sync.

(iv) Choose the 'GiftWrappingDataSource' to view its details and make sure the data source sync is complete from the Sync history section

![img]()

### 2. Edit the Agent

(i) Choose the Agent you created in the previous section of the workshop, and choose Edit in Agent Builder
(ii) Use the following as the updated instructions for the agent

_you are a product recommendations agent for gift products, the user is trying to buy a gift for someone and you are trying to help identify the best products based on the filters in the action groups, ask questions to identify at least one of the input filters, gender, category or occasion.
do not recommend any products that are not retrieved from the products API.
do not ask about the gender if it is obvious from the user input already.
Always start by getting the full list of products from the API so you can know the proper filter values to be used in the API parameters.
always use a single value for each filter field, and adhere to the filtration values based on the first API call.
And never tell the user about the API and its details.
After recommending products ask the user if they want to add any products to the cart, then use the add to cart API to add it, ask the user about his email and use it as user id and use it along the whole conversation and in any cart API calls, reply to the user with the user id after first cart addition to be used in later additions.
before item addition to the cart, use the get personalize recommendation API to get products other customer has bought based on the product that you just added to the cart, recommend that to the user by telling them that other customers also bought this along with the product that you just added and ask if they want to also add it to the cart as well.
after adding an item to the cart, retrieve the cart items from the get cart API and display it to the user, the user can ask about the items in the cart at any time, use the get cart api to answer that.
Before the end of the conversation ask the user if they want you to suggest gift wrapping ideas for the products in the cart, and get the wrapping ideas based on the gift wrapping knowledge base._

### 3. Add new Knowledge Base to Agent

(i) Navigate to the Knowledge Bases section under the Agent, then choose “Add” to add an Knowledge Base to enable the Agent to use the Gift Wrapping knowledge base.

(ii) Select the knowledge base 'GiftWrappingKnowledgeBase' and choose Action Group Type “Define with API Schemas”

(iii) Add the following instructions as the 'Knowledge base instructions for Agent'.

_This is a gift wrapping knowledge base for ideas on how to wrap gifts based on the gift details the occasion and its recipient_

(iv) Choose Add to add the knowledge base to the agent.

(v) Then on the Agent builder page, Choose “Save” and then “Prepare” on the top of the page. Now you can use the ‘Test Agent’ section on the right to start testing having conversations with the chatbot.


(vi) Test the Agent.

![img]()
![img]()
![img]()

- Proceed to next section to learn how to create specialized agents and collaborate between them
  
## Multi-Agent Collaboration
In this section, you will define two specialized agents that collaborate under a single supervisor agent to deliver an end-to-end shopping experience, showcasing multi-agent orchestration with Amazon Bedrock Agents. 
One agent, the Product Recommendation Agent, focuses on retrieving product information and suggestions,while the Cart Management Agent is responsible for cart actions such as adding items and fetching cart details.

### 1. Create a Product Recommendation Agent

(i) Create a new Agent from Amazon Bedrock Console > Agents , name it product-details-agent

(ii) Select Anthropic Claude Sonnet 4.5 as the model, deselect the 'Bedrock Agent Optimized' checkbox to see it

(iii) Select Use an existing service role and choose the role starting with producttableandapi-ws-IAMRole00AmazonBedrock.

(iv) Use the following as the instructions for the agent:

_you are a specialized product recommendation agent that focuses on getting product details.
Always start by getting the full list of products from the API so you can know the proper filter values to be used in the API parameters.
Help identify the best products based on the filters in the action groups by asking questions to identify at least one of the input filters: gender, category or occasion.
do not recommend any products that are not retrieved from the products API.
do not ask about the gender if it is obvious from the user input already.
always use a single value for each filter field, and adhere to the filtration values based on the first API call.
And never tell the user about the API and its details._

(v) In Additional Setting section Enable_ User input_

(vi) Add the get-product-recommendations action group as done in the previous sections, using the GetProductDetailsFunction Lambda function.

(vii) Choose "Save and exit"

(viii) Choose "Create Alias" to create an Alias for the agent, name it get-product-alias

   ![img]()
   
### 2. Create the Cart Management agent
(i) Create another Agent, name it cart-management-agent

(ii) Select Anthropic Claude Sonnet 4.5 as the model, Select Use an existing service role and choose the role starting with producttableandapi-ws-IAMRole00AmazonBedrock

(iii) Use the following as the instructions for the agent:

_you are a specialized cart management agent that handles cart operations.
you help users add items to their cart and retrieve cart details.
ask the user about their email to use as user id for cart operations if not provided.
after adding items to cart, always show the cart contents using the get cart API.
And never tell the user about the API and its details._

(iv) In Additional Setting section Enable _User input_

(v)Add both the add-item-to-cart and get-cart action groups as done in the previous sections.

(vi) Choose "Save and exit"

(vii) Choose "Create Alias" to create an Alias for the agent, name it cart-agent-alias


   ![img]()
   
### 3. Create the Supervisor Agent
(i) Create a new Agent to act as the supervisor, name it shopping-supervisor-agent

(ii) In the Multi-agent collaboration section, turn on the checkbox to Enable multi-agent collaboration. This is crucial as it identifies this agent as a supervisor that can coordinate with other agents.

(iii) Select _Anthropic Claude Sonnet 4.5_ as the model, Select Use an existing service role and choose the role starting with _producttableandapi-ws-IAMRole00AmazonBedrock_

(iv) Use the following as the instructions for the agent:

_you are a product recommendation agent, you help users get gifts for their loved ones.
you collaborate agents to do that, one agent helps get product details and recommendations call that first to know what are the product categories and occasions, get info about the products first and don't ask about any attribute that are not in the product information.
the other agent helps you to do actions on cart, to add product to cart and get cart details, respect exact product name while adding to cart._

(v) In Additional Setting section Enable _User input_

(vi) Now you can add both agents created above as collaborator agents in the Collaborator agents section.

   ![img]()
   
### 4. Configure Multi-Agent Collaboration
After creating the supervisor agent, you need to properly configure how it will collaborate with the specialized agents.

#### Add the Product Recommendation Collaborator
(i) In the supervisor agent configuration, navigate to the Multi-gent collaboration section and choose "Edit".

(ii) Expand the Agent Collaborator section, and from the Collaborator agent dropdown, select the product-details-agent and its latest alias

(iii) For Collaborator name, _enter product-recommender_

(iv) In the Collaboration instructions field, enter:

_Invoke this agent when the user needs product recommendations or information about available products. This agent should be called first to understand the product catalog and get recommendations based on user preferences._

(v) Enable Enable conversation history to allow the supervisor to share context from previous conversations. This helps maintain a coherent conversation flow when discussing products.

#### Add the Cart Management Collaborator
(i) Click Add collaborator

(ii) From the Collaborator agent dropdown, select the cart-management-agent and its latest alias

(iii) For Collaborator name, enter cart-manager

(iv) In the Collaboration instructions field, enter

_Invoke this agent when the user wants to add items to their cart or view cart contents. This agent should be called after the user has selected products they want to purchase._

(v) Enable Enable conversation history to ensure the cart manager understands which products the user selected during their conversation with the product recommender.

(vi) Choose Save and exit at the top of the page to add the agents to your collaboration team.

(vii) Then on the Agent builder page, Choose “Save” and then “Prepare” on the top of the page.

(viii) Test the Agent.

![img]()
![img]()
![img]()


#### Cleanup:
1. Go to the Agents section in Amazon Bedrock console, and delete the Agents you created.
2. Go to the AWS CloudFormation console, and delete the Stacks 'BedrockKB' and 'producttableandapi-ws'. 

 
## Conclusion
Congratulations! Built a smart shopping chatbot with Amazon Bedrock Agents: captures preferences, queries APIs for personalized product recs, adds cart ops, Personalize upsells, and KB ideas.Multi-agent collab—recs and cart specialists under a supervisor—powers scalable e-commerce.

## Reference
Amazon Bedrock Workshop: https://catalog.workshops.aws/e-commerce-in-a-bot/en-US/multi-agent-collaboration

