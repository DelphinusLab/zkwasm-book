# Typescript SDK for connecting with ZKWASM cloud.

## Introduction
zkWasm-service-helper(https://github.com/DelphinusLab/zkWasm-service-helper) is a typescript library to help communicate with zkwasm cloud service. It provides common APIs for ZKWASM application developers to use the cloud proving service to generate ZKWASM proofs.


## How to use it
The default RPC point for ZKWASM cloud is "https://rpc.zkwasmhub.com:8090". This lib main provide a ZkWasmServiceHelper class to help user to communicate to this RPC endpoint. It mainly provides the following APIs to perform proving tasks and query information about submitted tasks.

Example APIs:
* async queryImage(md5: string);
* async loadTasks(query: QueryParams);
* async addNewWasmImage(task: WithSignature<AddImageParams>);
* async addProvingTask(task: WithSignature<ProvingParams>);

### An example to add new wasm image
This example typescript code will add the wasm image to the zkwasm service.
```
import {
  AddImageParams,
  WithSignature,
  ZkWasmUtil,
  zkWasmServiceHelper
} from "zkwasm-service-helper";

const endpoint = ""https://rpc.zkwasmhub.com:8090";
let helper = new ZkWasmServiceHelper(endpoint, "", "");
let imagePath = "/home/user/a.wasm";
let fileSelected: Buffer = fs.readFileSync(imagePath);
let md5 = ZkWasmUtil.convertToMd5(
        fileSelected as Uint8Array
      );

let info: AddImageParams = {
        name: fileSelected.name,
        image_md5: md5,
        image: fileSelected,
        user_address: account!.address.toLowerCase(),
        description_url: "",
        avator_url: "",
        circuit_size: circuitSize,
      };

let msg = ZkWasmUtil.createAddImageSignMessage(info);
let signature: string = await ZkWasmUtil.signMessage(msgString, priv); //Need user private key to sign the msg
let task: WithSignature<AddImageParams> = {
        ...info,
        signature,
      };

let response = await helper.addNewWasmImage(task);

```

### An example to add proving tasks
This example typescript code will add proving tasks to the zkwasm service.
```
import {
  ProvingParams,
  WithSignature,
  ZkWasmUtil,
  zkWasmServiceHelper
} from "zkwasm-service-helper";

const endpoint = ""https://rpc.zkwasmhub.com:8090";
const image_md5 = "xxxx";
const public_inputs = "0x22:i64 0x21:i64";
const private_inputs = "";
const user_addr = "0xaaaaaa";

let helper = new ZkWasmServiceHelper(endpoint, "", "");
let pb_inputs: Array<string> = helper.parseProvingTaskInput(public_inputs);
let priv_inputs: Array<string> = helper.parseProvingTaskInput(private_inputs);

let info: ProvingParams = {
  user_address: user_addr.toLowerCase(),
  md5: image_md5,
  public_inputs: pb_inputs,
  private_inputs: priv_inputs,
};
let msgString = ZkWasmUtil.createProvingSignMessage(info);

let signature: string;
try {
  signature = await ZkWasmUtil.signMessage(msgString, priv);
} catch (e: unknown) {
  console.log("error signing message", e);
  return;
}

let task: WithSignature<ProvingParams> = {
  ...info,
  signature: signature,
};

let response = await helper.addProvingTask(task);
```

### An example to query task details
This example typescript code will query task details:
```
import {
    ZkWasmServiceHelper,
    ZkWasmUtil,
    QueryParams,
    PaginationResult,
    Task,
} from "zkwasm-service-helper";
import BN from "bn.js";

const endpoint = ""https://rpc.zkwasmhub.com:8090";
const taskid = "xxxx"

let helper = new ZkWasmServiceHelper(endpoint, "", "");
    let args: QueryParams = {
        id: taskid!,
        user_address: "",
        md5: "",
        tasktype: "",
        taskstatus: "",
    };
    helper.loadTasks(args).then((res) => {
        const tasks = res as PaginationResult<Task[]>;
        const task: Task = tasks.data[0];
        let aggregate_proof = ZkWasmUtil.bytesToBN(task.proof);
        let instances = ZkWasmUtil.bytesToBN(task.instances);
        let batchInstances = ZkWasmUtil.bytesToBN(task.batch_instances);
        let aux = ZkWasmUtil.bytesToBN(task.aux);
        let fee = task.task_fee && ZkWasmUtil.convertAmount(task.task_fee); 

        console.log("Task details: ");
        console.log("    ", task);
        console.log("    proof:");
        aggregate_proof.map((proof: BN, index) => {
            console.log("   0x", proof.toString("hex"));
        });
        console.log("    batch_instacne:");
        batchInstances.map((ins: BN, index) => {
            console.log("   0x", ins.toString("hex"));
        });
        console.log("    instacne:");
        instances.map((ins: BN, index) => {
            console.log("   0x", ins.toString("hex"));
        });
        console.log("    aux:");
        aux.map((aux: BN, index) => {
            console.log("   0x", aux.toString("hex"));
        });
        console.log("   fee:", fee);
    }).catch((err) => {
        console.log("queryTask Error", err);
    }).finally(() =>
        console.log("Finish queryTask.")
    );
```

### Notes:
md5 is case insensitive when communicate with our zkwasm service

