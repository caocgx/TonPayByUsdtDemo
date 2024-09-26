<template>
  <div class="main">
    <div class="container">
      <div class="connectBtn" @click="connectWallet">{{ btnText }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { TonConnectUI } from "@tonconnect/ui";
import { Address, beginCell, toNano, TonClient, JettonMaster } from "@ton/ton";
import { getHttpEndpoint } from "@orbs-network/ton-access";

const tonConnectUI = new TonConnectUI({
  manifestUrl:
    "https://test-genleap-ton.smilecobra.io/web-mobile/tonconnect-manifest.json",
});
const btnText = ref("连接钱包");
const tonClient = ref(null);
console.log(tonConnectUI.connected);

onMounted(async () => {
  const endpoint = await getHttpEndpoint({ network: "mainnet" });
  tonClient.value = new TonClient({
    endpoint: endpoint,
  });
});

async function connectWallet() {
  if (!tonConnectUI.connected) {
    console.log("未连接钱包");
    await tonConnectUI.openModal();
  } else {
    console.log("钱包已连接");

    const { address } = tonConnectUI.wallet.account;

    const destinationAddress = Address.parse(
      //接收地址
      "UQB5xx5Ve1FT2-0rQ9waV8S1_j1-xDoMUbc-dkZyIM_5S_SP"
    );
    const userAddress = Address.parse(address); //用户地址（调用钱包地址）
    const usdtMasterAddress = Address.parse(
      //USDT合约地址
      "EQCxE6mUtQJKFnGfaROTKOt1lZbDiiX1kCixRv7Nw2Id_sDs"
    );

    const usdtMaster = tonClient.value.open(
      JettonMaster.create(usdtMasterAddress)
    );
    const usdtWallet = await usdtMaster.getWalletAddress(userAddress);
    console.log("usdtWallet", usdtWallet.toString());

    const body = beginCell()
      .storeUint(0xf8a7ea5, 32)
      .storeUint(0, 64)
      .storeCoins(toNano("0.1"))
      .storeAddress(destinationAddress)
      .storeAddress(userAddress)
      .storeUint(0, 1)
      .storeCoins(toNano("0.0001"))
      .storeUint(0, 1)
      .endCell();

    const transaction = {
      validUntil: Math.floor(Date.now() / 1000) + 360,
      messages: [
        {
          address: usdtWallet.toString(),
          amount: toNano("0.0005").toString(),
          payload: body.toBoc().toString("base64"),
        },
      ],
    };

    console.log(transaction);

    const result = await tonConnectUI.sendTransaction(transaction);
  }
}
</script>


<style scoped>
.main {
  width: 100%;
  height: 100%;
  background: #1f1f1f;
}

.container {
  width: 100%;
  height: 100%;

  position: relative;
}

.connectBtn {
  position: absolute;
  left: 50%;
  bottom: 20px;
  transform: translateX(-50%);
  padding: 20px;
  width: 80%;
  background: #07988a;
  cursor: pointer;
  color: #fff;
  font-size: 14px;
  letter-spacing: 2px;
  border-radius: 8px;

  display: flex;
  align-content: center;
  justify-content: center;
}
</style>
