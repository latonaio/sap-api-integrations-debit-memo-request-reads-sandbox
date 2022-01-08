# sap-api-integrations-debit-memo-request-reads
sap-api-integrations-debit-memo-request-reads は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で デビットメモ依頼データを取得するマイクロサービスです。    
sap-api-integrations-debit-memo-request-reads には、サンプルのAPI Json フォーマットが含まれています。   
sap-api-integrations-debit-memo-request-reads は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。   
https://api.sap.com/api/OP_API_DEBIT_MEMO_REQUEST_SRV_0001/overview  

## 動作環境  
sap-api-integrations-debit-memo-request-reads は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）　　

## クラウド環境での利用
sap-api-integrations-debit-memo-request-reads は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。  

## 本レポジトリ が 対応する API サービス
sap-api-integrations-debit-memo-request-reads が対応する APIサービス は、次のものです。

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_DEBIT_MEMO_REQUEST_SRV_0001/overview   
* APIサービス名(=baseURL): API_DEBIT_MEMO_REQUEST_SRV

## 本レポジトリ に 含まれる API名
sap-api-integrations-sales-order-reads には、次の API をコールするためのリソースが含まれています。  

* A_DebitMemoRequest（デビットメモ依頼 - ヘッダ）※デビットメモ依頼の詳細データを取得するために、ToHeaderPartner、ToItem、ToItemPricingElement、と合わせて利用されます。
* A_DebitMemoRequestItem（デビットメモ依頼 - 明細）※デビットメモ依頼明細の詳細データを取得するために、ToItemPricingElement、と合わせて利用されます。
* ToHeaderPartner（デビットメモ依頼 - ヘッダ取引先）
* ToItem（デビットメモ依頼 - 明細）
* ToItemPricingElement（デビットメモ依頼 - 明細価格条件）

## API への 値入力条件 の 初期値
sap-api-integrations-sales-order-reads において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.DebitMemoRequest.DebitMemoRequest（デビットメモ依頼）
* inoutSDC.DebitMemoRequest.DebitMemoRequestItem（デビットメモ依頼明細）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"Header" が指定されています。

```
	"api_schema": "sap.s4.beh.debitmemorequest.v1.DebitMemoRequest.Created.v1",
	"accepter": ["Header"],
	"debit_memo_request": "70000000",
	"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
	"api_schema": "sap.s4.beh.debitmemorequest.v1.DebitMemoRequest.Created.v1",
	"accepter": ["All"],
	"debit_memo_request": "70000000",
	"deleted": false
```

## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetDebitMemoRequest(debitMemoRequest, debitMemoRequestItem string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "Header":
			func() {
				c.Header(debitMemoRequest)
				wg.Done()
			}()
		case "Item":
			func() {
				c.Item(debitMemoRequest, debitMemoRequestItem)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```
## Output  
本マイクロサービスでは、[golang-logging-library](https://github.com/latonaio/golang-logging-library) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP デビットメモ依頼 の ヘッダデータ が取得された結果の JSON の例です。  
以下の項目のうち、"DebitMemoRequest" ～ "to_Item" は、/SAP_API_Output_Formatter/type.go 内 の Type Header {} による出力結果です。"cursor" ～ "time"は、golang-logging-library による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-debit-memo-request-reads/SAP_API_Caller/caller.go#L58",
	"function": "sap-api-integrations-debit-memo-request-reads/SAP_API_Caller.(*SAPAPICaller).Header",
	"level": "INFO",
	"message": [
		{
			"DebitMemoRequest": "70000000",
			"DebitMemoRequestType": "DMR1",
			"SalesOrganization": "1010",
			"DistributionChannel": "10",
			"OrganizationDivision": "00",
			"SalesGroup": "",
			"SalesOffice": "",
			"SalesDistrict": "",
			"SoldToParty": "10100001",
			"CreationDate": "/Date(1480982400000)/",
			"LastChangeDate": "",
			"LastChangeDateTime": "/Date(1481036013881+0000)/",
			"PurchaseOrderByCustomer": "PO 4500000467",
			"CustomerPurchaseOrderType": "",
			"CustomerPurchaseOrderDate": "/Date(1480464000000)/",
			"DebitMemoRequestDate": "/Date(1480982400000)/",
			"TotalNetAmount": "560.00",
			"TransactionCurrency": "EUR",
			"SDDocumentReason": "",
			"PricingDate": "/Date(1480982400000)/",
			"CustomerTaxClassification1": "",
			"HeaderBillingBlockReason": "",
			"IncotermsClassification": "EXW",
			"CustomerPaymentTerms": "0004",
			"PaymentMethod": "",
			"BillingDocumentDate": "/Date(1480982400000)/",
			"ReferenceSDDocument": "23212",
			"ReferenceSDDocumentCategory": "C",
			"OverallSDProcessStatus": "A",
			"TotalCreditCheckStatus": "",
			"OverallSDDocumentRejectionSts": "A",
			"OverallOrdReltdBillgStatus": "A",
			"to_Partner": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_DEBIT_MEMO_REQUEST_SRV/A_DebitMemoRequest('70000000')/to_Partner",
			"to_Item": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_DEBIT_MEMO_REQUEST_SRV/A_DebitMemoRequest('70000000')/to_Item"
		}
	],
	"time": "2022-01-08T16:46:02.200391+09:00"
}
```


