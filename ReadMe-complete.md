Task: Relay Data Between Two Hyperledger Fabric Networks Using Hyperledger Cacti

In this guide, you will learn how to relay data between two Hyperledger Fabric networks (e.g., Network A and Network B) using Hyperledger Cacti. The task involves setting up Cacti to handle cross-network communication, modifying chaincode on both networks, and implementing relay logic in Cacti.


---

Prerequisites

1. Two Hyperledger Fabric Networks: Ensure both networks (Network A and Network B) are up and running.


2. Hyperledger Cacti: Install and configure Cacti to act as the intermediary between the networks.


3. Go Language Setup: The chaincode for both networks is written in Go.


4. Node.js Setup: For the relay logic using Cacti, ensure Node.js is installed.




---

Step 1: Set Up Hyperledger Cacti

Start by setting up Hyperledger Cacti to facilitate communication between the two networks.

1.1 Clone the Cacti Repository

git clone https://github.com/hyperledger/cacti.git
cd cacti

1.2 Install Cacti Dependencies

cd ./packages/cactus-plugin-ledger-connector-fabric-socketio/
npm install

1.3 Set Up Fabric Connectors

Set up two Fabric connectors in the Cacti configuration—one for Network A and one for Network B. Here’s an example configuration for Network A:

{
  "pluginRegistry": [
    {
      "packageName": "@hyperledger/cacti-plugin-ledger-connector-fabric",
      "pluginInstanceId": "plugin-connector-fabric-networkA",
      "options": {
        "connectionProfile": "path_to_networkA_connection_profile.json",
        "username": "admin",
        "channelName": "mychannel",
        "chainCodeId": "fabcar",
        "organization": "Org1MSP",
        "logLevel": "debug"
      }
    }
  ]
}

You will also need a similar configuration for Network B.


---

Step 2: Modify Chaincode for Event Emission and Data Relaying

Next, you’ll need to modify the chaincode on both networks to support event emission (on Network A) and data relaying (on Network B).

2.1 Chaincode for Network A (Fabcar)

Here is an example of the modified Go chaincode for Network A to emit events when a new car is created:

package main

import (
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type SmartContract struct {
	contractapi.Contract
}

type Car struct {
	Make   string `json:"make"`
	Model  string `json:"model"`
	Color  string `json:"color"`
	Owner  string `json:"owner"`
}

func (s *SmartContract) InitLedger(ctx contractapi.TransactionContextInterface) error {
	cars := []Car{
		{Make: "Toyota", Model: "Prius", Color: "blue", Owner: "Tomoko"},
		{Make: "Ford", Model: "Mustang", Color: "red", Owner: "Brad"},
	}

	for i, car := range cars {
		carAsBytes, _ := json.Marshal(car)
		err := ctx.GetStub().PutState("CAR"+strconv.Itoa(i), carAsBytes)
		if err != nil {
			return fmt.Errorf("Failed to put car in ledger: %v", err)
		}
	}
	return nil
}

func (s *SmartContract) CreateCar(ctx contractapi.TransactionContextInterface, carID string, make string, model string, color string, owner string) error {
	car := Car{
		Make:  make,
		Model: model,
		Color: color,
		Owner: owner,
	}

	carAsBytes, _ := json.Marshal(car)
	err := ctx.GetStub().PutState(carID, carAsBytes)
	if err != nil {
		return fmt.Errorf("Failed to add car to ledger: %v", err)
	}

	// Emit event for Cacti to pick up
	err = ctx.GetStub().SetEvent("CarCreated", carAsBytes)
	if err != nil {
		return fmt.Errorf("Failed to emit event: %v", err)
	}

	return nil
}

func (s *SmartContract) QueryCar(ctx contractapi.TransactionContextInterface, carID string) (*Car, error) {
	carAsBytes, err := ctx.GetStub().GetState(carID)
	if err != nil {
		return nil, fmt.Errorf("Failed to read from world state: %v", err)
	}
	if carAsBytes == nil {
		return nil, fmt.Errorf("Car %s does not exist", carID)
	}

	var car Car
	err = json.Unmarshal(carAsBytes, &car)
	if err != nil {
		return nil, err
	}

	return &car, nil
}

This code emits a CarCreated event when a new car is added to Network A, which Cacti will listen for.

2.2 Chaincode for Network B (Fabcar)

The chaincode for Network B needs to support receiving data relayed from Network A. Here is an example:

func (s *SmartContract) RelayCar(ctx contractapi.TransactionContextInterface, carID string, carDetails string) error {
	var car Car
	err := json.Unmarshal([]byte(carDetails), &car)
	if err != nil {
		return fmt.Errorf("Error unmarshalling car data: %v", err)
	}

	carAsBytes, _ := json.Marshal(car)
	err = ctx.GetStub().PutState(carID, carAsBytes)
	if err != nil {
		return fmt.Errorf("Failed to store relayed car: %v", err)
	}

	return nil
}

This function accepts a car’s details (relayed from Network A) and stores it in Network B’s ledger.


---

Step 3: Implement the Cacti Relay Logic

Now, you will implement the relay logic in Hyperledger Cacti to listen for events from Network A and relay data to Network B.

3.1 Relay Logic in Cacti (Node.js)

Here’s the relay logic using Cacti connectors to listen for a CarCreated event from Network A and send the car data to Network B:

const { PluginLedgerConnectorFabric } = require("@hyperledger/cacti-plugin-ledger-connector-fabric");

async function relayCarFromNetworkAtoNetworkB(networkAClient, networkBClient, carID) {
    try {
        // Query the car data from Network A
        const carData = await networkAClient.transact({
            contractName: 'fabcar',
            channelName: 'mychannel',
            methodName: 'QueryCar',
            args: [carID],
        });
        console.log(`Car data from Network A: ${JSON.stringify(carData)}`);

        // Relay the car data to Network B
        const relayResponse = await networkBClient.transact({
            contractName: 'fabcar',
            channelName: 'mychannel',
            methodName: 'RelayCar',
            args: [carID, JSON.stringify(carData)],
        });
        console.log(`Data successfully relayed to Network B: ${relayResponse}`);
    } catch (error) {
        console.error("Error relaying car data: ", error);
    }
}

// Event listener for Network A
networkAClient.on("CarCreated", async (event) => {
    console.log("CarCreated event detected from Network A", event);
    const carID = event.key;
    await relayCarFromNetworkAtoNetworkB(networkAClient, networkBClient, carID);
});

Step 4: Final Considerations

1. Authentication and Security: Ensure that both networks have secure authentication set up (admin credentials, secure connection profiles) and that Cacti connectors are properly authenticated.


2. Event Listening: The event listener should be optimized to listen for events in real-time and relay data with minimal delay.


3. Error Handling: Include robust error handling to retry failed transactions and log errors.


4. Scalability: If dealing with large amounts of data, consider batching transactions or using asynchronous processing.




---

Conclusion

This guide has demonstrated how to relay data between two Hyperledger Fabric networks using Hyperledger Cacti. By modifying the chaincode to emit and relay data and using Cacti for cross-network communication, you can achieve seamless integration between distributed ledger systems.

