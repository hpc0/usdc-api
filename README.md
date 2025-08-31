# **USDC Indexer and API**

This project provides a robust API for querying USDC token transfers on the Ethereum blockchain. It features a self-contained indexing service that listens for and records Transfer events, making the data available through a set of REST endpoints.

## **1\. Setup Instructions**

To run this application, you will need to have Docker and Docker Compose installed.

### **Prerequisites**

* [Docker](https://www.docker.com/products/docker-desktop)  
* [Docker Compose](https://docs.docker.com/compose/install/)

### **Step-by-Step Guide**

1. Create the environment file.  
   Create a new file named .env.docker in the root of your project with the following content. This file will be used by the Docker container at runtime.
   
   DATABASE\_URL=postgresql://usdc:usdc@usdc_db:5432/usdc\_indexer  
   ETH\_RPC\_URL=https://mainnet.infura.io/v3/\<YOUR\_API\_KEY\>  
   USDC\_CONTRACT=0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48  
   MAX\_BLOCKS=3  
   MAX\_INDEXER\_SIZE=1000  

   **Note:** You must replace \<YOUR\_API\_KEY\> with a valid infura api key.  
3. Run the application.  
   Use the following command to create a fresh container and volume.  
   docker-compose down \-v && docker-compose up \--build


   The docker-compose up \--build
   

   * Build the usdc\_app Docker image.  
   * Start the usdc\_db and usdc\_app containers.  
   * Start the API server at http://localhost:3000.  

   
4. Access the API.  
   Once the containers are up and running, you can access the API at http://localhost:3000. An interactive API documentation page is available at http://localhost:3000/api.

## **2\. API Documentation**

### **Get recent transfers**

* GET /transfers  
* **Description:** Fetches a list of recent USDC transfers from the database.  
* **Query Parameters:**  
  * from (optional): Filter transfers by sender address.  
  * to (optional): Filter transfers by recipient address.  
  * limit (optional): Maximum number of transfers to return (default: 20).

### **Get address balance**

* GET /transfers/balance/:address  
* **Description:** Fetches the real-time USDC balance of a given Ethereum address by calling the blockchain RPC.  
* **Path Parameters:**  
  * address (required): The Ethereum address to query.

### **Get transfer history**

* GET /transfers/history/:address  
* **Description:** Fetches a list of recent transfers where the given address was either the sender or the recipient.  
* **Path Parameters:**  
  * address (required): The Ethereum address to fetch history for.  
* **Query Parameters:**  
  * limit (optional): Maximum number of transfers to return (default: 20).

### **Simulate a transfer (Demo)**

* POST /transfers/transfer  
* **Description:** This endpoint simulates a USDC transfer to demonstrate how a transaction would be constructed. It does **not** broadcast an actual transaction to the blockchain.  
* **Request Body:**  
  * fromPk (required): The private key of the sender's wallet.  
  * to (required): The recipient's Ethereum address.  
  * amount (required): The amount of USDC to transfer (e.g., "100.5").

## **3\. Fault Tolerance Approach**

* **Automatic Recovery:** The system is designed to perform the re-indexing process always by fetching from the latest known data, ensuring that an interrupted fetch does not lead to a gap in your data. It is not a problem if a previous fetch was interrupted.

* **Data Integrity:** The indexer prioritizes data integrity by first successfully fetching and writing newer records to the database. Only after this new data is confirmed as stored does the system delete older records to maintain the configured index size.
