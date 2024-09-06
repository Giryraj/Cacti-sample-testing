# Cacti-sample-testing

Implementing **Hyperledger Cacti** to relay data between two **Hyperledger Fabric** networks involves multiple steps. The following guide will take you through the process of setting up Hyperledger Cacti, configuring your networks, and writing code for relaying data between the networks.

### Prerequisites

1. **Hyperledger Fabric Networks:** Ensure both networks (let's call them `Network A` and `Network B`) are fully operational.
2. **Docker & Docker Compose**: Required to run Cacti and related containers.
3. **Node.js**: Used for the Cacti SDK and server.
4. **Git**: For cloning repositories.
5. **Hyperledger Cacti**: The interoperation framework between multiple DLTs.

### Steps to Implement Hyperledger Cacti for Cross-Network Communication

#### 1. Clone Hyperledger Cacti Repository
Start by cloning the **Cacti** repository from GitHub and navigate to the directory:
```bash
git clone https://github.com/hyperledger/cacti.git
cd cacti
```

#### 2. Set up the Cacti Environment
Install dependencies for running Hyperledger Cacti. 

```bash
cd ./packages/cactus-plugin-ledger-connector-fabric-socketio/
npm install
```

#### 3. Set up Configuration for Hyperledger Fabric Connectors
You'll need to configure Cacti to interact with both `Network A` and `Network B`. Hyperledger Cacti uses **ledger connectors** to communicate with various blockchain networks.

- Navigate to the Fabric connector configuration files and edit the settings to match the details of your Fabric networks.

For example, create a configuration file for `Network A`:
```json
{
  "pluginRegistry": [
    {
      "packageName": "@hyperledger/cactus-plugin-ledger-connector-fabric",
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
```

Similarly, create another configuration for `Network B`.

#### 4. Set Up Connection Profiles for Each Fabric Network

Create the connection profiles for both Fabric networks.

For **Network A**, the connection profile might look like:

```json
{
  "name": "networkA",
  "version": "1.0.0",
  "client": {
    "organization": "Org1",
    "connection": {
      "timeout": {
        "peer": {
          "endorser": "300"
        }
      }
    }
  },
  "channels": {
    "mychannel": {
      "orderers": ["orderer.example.com"],
      "peers": {
        "peer0.org1.example.com": {}
      }
    }
  },
  "organizations": {
    "Org1": {
      "mspid": "Org1MSP",
      "peers": ["peer0.org1.example.com"],
      "certificateAuthorities": ["ca.org1.example.com"]
    }
  },
  "orderers": {
    "orderer.example.com": {
      "url": "grpcs://localhost:7050"
    }
  },
  "peers": {
    "peer0.org1.example.com": {
      "url": "grpcs://localhost:7051"
    }
  }
}
```

Similarly, create the connection profile for **Network B**.

#### 5. Set Up Docker Containers for Cacti

After configuring the connection profiles, start the Cacti services using Docker. Hyperledger Cacti has pre-configured Docker images for running its services.

```bash
docker-compose -f ./tools/docker/fabric-socketio/docker-compose.yaml up
```

This will launch the Cacti server and connectors.

#### 6. Develop Relay Logic

You'll need to write logic to relay data from one network to the other. This is typically done through **smart contracts** and **Cacti plugins**.

Example of a Cacti relay function in Node.js:
```javascript
const FabricConnector = require('@hyperledger/cacti-plugin-ledger-connector-fabric');

async function relayData(networkAClient, networkBClient, data) {
    // Send data to Network A
    const txA = await networkAClient.transact({
        contractName: 'fabcar',
        channelName: 'mychannel',
        methodName: 'createCar',
        args: data,
    });

    console.log("Data sent to Network A: ", txA);

    // Relay same data to Network B
    const txB = await networkBClient.transact({
        contractName: 'fabcar',
        channelName: 'mychannel',
        methodName: 'createCar',
        args: data,
    });

    console.log("Data relayed to Network B: ", txB);
}
```

In this function:
- `networkAClient` and `networkBClient` are Cacti Fabric connectors that interact with `Network A` and `Network B` respectively.
- `data` is the information being relayed between the networks.

#### 7. Test the Setup

Now that the relay logic is set up, you can test cross-chain communication.

1. **Deploy Smart Contracts**: Ensure the `fabcar` contract is deployed on both networks (or a custom contract for your use case).
2. **Invoke Transactions**: Use Cacti to submit transactions to both networks.
3. **Verify Data Relay**: Ensure the data has been successfully relayed between the networks.

For example, running the relay:
```javascript
const networkAClient = new FabricConnector({
    connectionProfile: 'path_to_networkA_connection_profile.json',
    username: 'admin',
    channelName: 'mychannel',
    chainCodeId: 'fabcar',
    organization: 'Org1MSP',
});

const networkBClient = new FabricConnector({
    connectionProfile: 'path_to_networkB_connection_profile.json',
    username: 'admin',
    channelName: 'mychannel',
    chainCodeId: 'fabcar',
    organization: 'Org1MSP',
});

relayData(networkAClient, networkBClient, ["CAR10", "Toyota", "Prius", "White", "Tom"]);
```

#### 8. Monitor and Debug

Use logs and transaction results from Cacti and Fabric networks to monitor the system. Ensure proper error handling for network failures or transaction errors.

### Final Considerations

- **Security**: Ensure that all communication between Cacti, the Fabric networks, and the smart contracts are secured using proper credentials (TLS, private keys).
- **Network Configuration**: For a production environment, configure network policies for high availability and scalability.

### Conclusion

By following these steps, you will be able to use **Hyperledger Cacti** to relay data between two **Hyperledger Fabric** networks. The approach can be customized based on the specific data formats and contract logic you use.
