# 跨链环境使用说明
## 主链环境
	rpc：138.91.6.125：20336

## 子链环境
	rpc：138.91.6.125：30336   sideChainId = 3092255979

## 资产转移流程：
	用户在主链冻结ong，子链自动会生成对应数量的ongx（根据ong/ongx兑换比例）。
	用户在子链销毁ongx，子链自动会解锁对应数量ong（根据ongx/ong兑换比例）。
  用户只需调用主链的冻结ong接口和子链的销毁ongx接口即可完成资产的转移。

## 主链的冻结ong接口调用方法：
"""java
    public String ongSwap(Account account, SwapParam param, Account payer, long gaslimit, long gasprice) throws Exception {
        if(account == null || param == null || payer == null || gaslimit < 0|| gasprice < 0){
            throw new SDKException(ErrorCode.OtherError("parameter is wrong"));
        }
        List list = new ArrayList();
        Struct struct = new Struct();
        struct.add(param.sideChainId, param.address, param.ongXAccount);
        list.add(struct);
        byte[] args = NativeBuildParams.createCodeParamsScript(list);
        Transaction tx = sdk.vm().buildNativeParams(new Address(Helper.hexToBytes(contractAddress)),"ongSwap",args,payer.getAddressU160().toBase58(),gaslimit, gasprice);
        sdk.signTx(tx,new Account[][]{{account}});
        if(!account.equals(payer)){
            sdk.addSign(tx,payer);
        }
        boolean b = sdk.getConnect().sendRawTransaction(tx.toHexString());
        if (b) {
            return tx.hash().toString();
        }
        return null;
    }
"""
github地址：https://github.com/lucas7788/ontology-java-sdk/blob/sidechainsdk/src/main/java/com/github/ontio/smartcontract/nativevm/SideChainGovernance.java

## 子链的销毁ongx接口调用方法：
"""java
    public String ongxSwap(Account account, Swap swap, Account payer, long gaslimit, long gasprice) throws Exception {
        if(account == null || swap == null|| swap.value <=0 || payer == null || gaslimit < 0||gasprice < 0){
            throw new SDKException(ErrorCode.ParamError);
        }
        List list = new ArrayList();
        Struct struct = new Struct();
        struct.add(swap.address, swap.value);
        list.add(struct);
        byte[] args = NativeBuildParams.createCodeParamsScript(list);
        Transaction tx = sdk.sidechainVm().buildNativeParams(new Address(Helper.hexToBytes(ongXContract)),"ongxSwap",args,payer.getAddressU160().toBase58(),gaslimit, gasprice);
        sdk.sidechainVm().addSign(tx, account);
        sdk.getSideChainConnectMgr().sendRawTransaction(tx.toHexString());
        return tx.hash().toHexString();
    }
"""
github地址：https://github.com/lucas7788/ontology-java-sdk/blob/sidechainsdk/src/main/java/com/github/ontio/sidechain/smartcontract/ongx/OngX.java
