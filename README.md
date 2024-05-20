# hardhat-deploy-params

Setup contract deploy parameters in hardhat config file.

#### 1. Full example

```javascript
import {
  extendConfig,
  extendEnvironment,
  HardhatUserConfig,
} from "hardhat/config";
import "hardhat/types/config";
import "hardhat/types/runtime";
import { HardhatConfig } from "hardhat/types";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: "0.8.25",
  deployParams: {  // <--- Position 1
    localhost: {
      SCRAddress: "",
      USDTAddress: "",
    },
    polygon: {
      SCRAddress: "",
      USDTAddress: "",
    },
    amoy: {
      SCRAddress: "0xdC907cd32Bc3D6bb2c63Ede4E28c3fAcdd1d5189",
      USDTAddress: "0xca152522f26811fF8FcAf967d4040F7C6BbF8eaA",
    },
  },
};

interface DeployParam { // <--- Position 2
  SCRAddress: string;
  USDTAddress: string;
}

// <--- Position 3
//
// ===== ===== deployParams Plugin ===== =====
// How to use?
//
// ```
// const dp: DeployParam = hre.deployParam;
// ```
declare module "hardhat/types/config" {
  export interface DeployParamsConfig {
    [networkName: string]: DeployParam;
  }

  export interface HardhatUserConfig {
    deployParams: DeployParamsConfig;
  }

  export interface HardhatConfig {
    deployParams: DeployParamsConfig;
  }
}
extendConfig(
  (config: HardhatConfig, userConfig: Readonly<HardhatUserConfig>) => {
    config.deployParams = userConfig.deployParams;
  },
);

declare module "hardhat/types/runtime" {
  export interface HardhatRuntimeEnvironment {
    deployParam: DeployParam;
  }
}
extendEnvironment((hre) => {
  hre.deployParam = (function (): DeployParam {
    const dp = hre.config.deployParams[hre.network.name];
    if (hre.network.name !== "hardhat" && dp == undefined) {
      throw new Error(
        `'deployParams' configuration not found for '${hre.network.name}' network`,
      );
    }
    return dp;
  })();
});
// ===== ===== deployParams Plugin ===== =====

export default config;
```

#### 3. Explanation

- `Position 1` is the place where you can setup your deploy parameters. You need to setup deploy parameters for each network.
- `Position 2` is the place where you can define the type of deploy parameters.
- `Position 3` is the place where the plugin is defined. This plugin will help you to access deploy parameters in your script.

#### 4. How to use

```javascript
const ScoreLendFactory = await hre.ethers.getContractFactory("ScoreLend");
const scoreLend = await hre.upgrades.deployProxy(ScoreLendFactory, [
    hre.deployParam.SCRAddress,
    hre.deployParam.USDTAddress,
]);
await scoreLend.waitForDeployment();
```

`hre.deployParam` property is `DeployParam` type.

This code will work properly with `npx hardhat run --netwokr <your_network> <your_script>` command's `--netwokr <your_network>` option. The value of `hre.deployParam` property is according to the network you are using.
