# Decodable - 여러 타입을 가지는 키 만들기

medium에 [[iOS] Codable을 이용하여 JSON decoding하기](https://medium.com/@adie0423/ios-codable%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-json-decoding%ED%95%98%EA%B8%B0-f2d4e4b932b7) 라는 포스팅을 작성한 적이 있었기 때문에 자신만만했지만 난관이 꽤 많았고 삽질이란 삽질은 다 했기 때문에 다시 정리하려고 한다.



위에서 언급한 포스팅에 작성한 것처럼 `convertFromSnakeCase` 를 사용했다.

```swift
decoder.keyDecodingStrategy = .convertFromSnakeCase
```



API 문서에 써 있는 JSON 구조는 다음과 같았다.

```json
// Response - success
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": {
        "version": "0.1a",
        "prev_block_hash": "48757af881f76c858890fb41934bee228ad50a71707154a482826c39b8560d4b",
        "merkle_tree_root_hash": "fabc1884932cf52f657475b6d62adcbce5661754ff1a9d50f13f0c49c7d48c0c",
        "time_stamp": 1516498781094429,
        "confirmed_transaction_list": [ 
            {
                "version": "0x3",
                "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
                "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
                "value": "0xde0b6b3a7640000",
                "stepLimit": "0x12345",
                "timestamp": "0x563a6cf330136",
                "nid": "0x3",
                "nonce": "0x1",
                "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA=",
                "txHash": "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238",
                "dataType": "call",
                "data": {
                    "method": "transfer",
                    "params": {
                        "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
                        "value": "0x1"
                    }
                }
            }
        ],
        "block_hash": "1fcf7c34dc875681761bdaa5d75d770e78e8166b5c4f06c226c53300cbe85f57",
        "height": 3,
        "peer_id": "hx86aba2210918a9b116973f3c4b27c41a54d5dafe",
        "signature": "MEQCICT8mTIL6pRwMWsJjSBHcl4QYiSgG8+0H3U32+05mO9HAiBOhIfBdHNm71WpAZYwJWwQbPVVXFJ8clXGKT3ScDWcvw=="
    }
}

```



그래서 이 문서에 맞춰서 아래와 같이  `Decodable` 모델을 구성했다.

```swift
open class ConfirmedTransactionList: Decodable {
	public var from: String
	public var to: String
	public var timestamp: String
	public var signature: String
	public var txHash: String

	public var version: String?
	public var nid: String?
	public var stepLimit: String?
	public var value: String?

	public var nonce: String?
	public var dataType: String?
	public var data: DataInfo?

    public var fee: String?
	public var method: String?
                
	open class DataInfo: Decodable {
		public var method: String?
	public var params: [String: String]?
	}
}
```



잘 되는 줄 알고 이제 예제를 만들어보려고 하는데 `Parsing error`가 떴다. :tired_face:

대체 왜…. 어제까지만 해도 잘 되던게… 안될까ㅠㅠㅠㅠ하고 그 문제가 일어나는 부분의 JSON을 보았더니 이런 식이었다.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "version": "0.1a",
        "prev_block_hash": "3141e30db0b7036cfd750d9105d90ef95289c3a6f7e896999a4997c68c002fb8",
        "merkle_tree_root_hash": "d5a7106af08f96871c7c2bd6d5e16d5159486aff73f8d4ef84ff35e32d8d6b39",
        "time_stamp": 1550764885497421,
        "confirmed_transaction_list": [
            {
                "from": "hx04bebefa2c8c8378c20393d8259afe38a414e160",
                "to": "hx605b5ddd50a4b0314b93bcd80561aa5d37d4b46f",
                "value": "0x3635c9adc5dea00000",
                "version": "0x3",
                "nid": "0x1",
                "stepLimit": "0x193e8",
                "timestamp": "0x5826996223350",
                "data": "0x49434f4e65782074696c204c6564676572",
                "dataType": "message",
                "signature": "jxD3/Pe1OfIrgfsPgxReWxMbhSdTc8inp7mhoOps6ItS71vracx8UZdOrwgkoS7sFV1F8+ldN3LNQd2BZ9CykwE=",
                "txHash": "0xd5a7106af08f96871c7c2bd6d5e16d5159486aff73f8d4ef84ff35e32d8d6b39"
            }
        ],
        "block_hash": "bfbb0ae7770247bb2c7ca6d3b3fbcff398f4f3bc8bd68b7d511bc5e1d8499297",
        "height": 202131,
        "peer_id": "hx7563e2514a0865630216903c5fd166ec0fdb217a",
        "signature": "eSem6PHyHWxWyOretx0PAX+xEYyC+G0iuoS0ooIziy5iEzW4XrliQrHY59igC4d8krszw9TJoAYUAGsViZsLOwA="
    },
    "id": 1234
}
```



`data` 라는 값이 두가지의 타입을 가져야하는 상황이었다.

그래서 갓구글에 검색해서 다음과 같은 답변을 찾아냈다.

 https://stackoverflow.com/a/47319012

 https://stackoverflow.com/a/50067514



enum을 이용하여 해결했다.

```swift
open class ConfirmedTransactionList: Decodable {
	public var from: String
	public var to: String
	public var timestamp: String
	public var signature: String
	public var txHash: String

	public var version: String?
	public var nid: String?
	public var stepLimit: String?
	public var value: String?

	public var nonce: String?
	public var dataType: String?
                
	public var data: DataValue?
                
	public var fee: String?
	public var method: String?
                
	public enum DataValue: Decodable {
		case string(String)
		case dataInfo(DataInfo)
                    
		public init(from decoder: Decoder) throws {
			if let string = try? decoder.singleValueContainer().decode(String.self) {
				self = .string(string)
				return
			}

			if let dataInfo = try? decoder.singleValueContainer().decode(DataInfo.self) {
				self = .dataInfo(dataInfo)
				return
			}
			throw DataValueError.missingValue
		}
    public enum DataValueError: Error {
        case missingValue
        }
    }
                
	open class DataInfo: Decodable {
		public var method: String?
		public var params: [String: String]?
	}
}
```



data에 있는 값 찾아오는 방법은 다음과 같다.

```swift
let val = value.result.confirmedTransactionList.first?.data

switch val {
	case .string(let data)?:
  	print("String: \(data)")
	case .dataInfo(let data)?:
		print("DataInfo: \(data.method) \(data.params)")
	case .none:
		print("None")
}
```

