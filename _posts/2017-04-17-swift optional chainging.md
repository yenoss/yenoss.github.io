---
layout: post
title:  "CHAPTER 14 optional chainging"
date:   2017-4-17 15:37:00
categories: Swift
---

Swift 
========




CHAPTER 14 optional chainging



//chapter 14
//옵셔널 체이닝은 nil 일지 모르는 옵셔널에 속해 있는 프로퍼티 등을 가져오는 과정. 
//

~~~
class Room {
    var number: Int
    init(number: Int){
        self.number = number
    }
}
class Building {
    var name: String
    var room: Room?
    init(name: String) {
        self.name = name
    }
}
//struct Address {
//    var province: String
//    var city: String
//    var street: String
//    var building: Building?
//    var detailAddress: String?
//}
struct Address{
    var province: String
    var city: String
    var street: String
    var building: Building?
    var detailAddress: String?
    //초기화
    init(province: String, city: String, street: String){
        self.province = province
        self.city = city
        self.street = street
    }
    //
    func fullAddress() ->String? {
        
        var restAddress: String? = nil
        
        //빌딩정보가 있으면 넣고 없으면 디테일 어드레스를 넣는다.
        if let buildingInfo: Building = self.building {
            restAddress = buildingInfo.name
        } else if let detail = self.detailAddress {
            restAddress = detail
        }
        //주소 쭉이어서 출력할수있도록 모아줌
        if let rest: String = restAddress {
            var fullAddress: String = self.province
            
            fullAddress += "" + self.city
            fullAddress += "" + self.street
            fullAddress += "" + rest
            return fullAddress
        } else {
            return nil
        }
        
    }
    func printAddress() {
        if let address: String = self.fullAddress() {
            print(address)
        }
    }
    
}


class Persons {
    var name: String
    var address: Address?
    init(name: String){
        self.name = name
    }
}



let pkk: Persons = Persons(name: "choipk")
//pkk.address =  Addresss(province: "korea", city: "seoul", street: "seoulong", building: nil, detailAddress: "detail0202", detailAddress: nil)
pkk.address = Address(province: "korea", city: "seoul", street: "anam")
pkk.address?.building = Building(name:"anam")
pkk.address?.building?.room = Room(number: 10)

pkk.address?.printAddress()
~~~


체인닝의 전과 후의 예제는 아래와 같다

~~~
let pkRoomViaOptionalUnwrap: Int = pkk.address!.building!.room!.number
var rommNum: Int? = nil
if let pkRooms: Addresss = pkk.addresss {
    if let pkBuilding: Building = pkRooms.building {
        if let pkRoom: Room = pkBuildingg.room {
            rommNum = pkRoom.number
        }
    }
}

if let number: Int = rommNum {
    print(number)
}else{
    print("cant find room num")
}

//위와 같이 표현할 수 있지만 너무길다.
///아래와같이 표현할 수 있다.
//?를통해 nil이 아님을 확인함과 동시에 값을 넣을 수 있다.

if let roomNumbersShort = pkk.address?.building?.room?.number{
    print("number => \(roomNumbersShort)")
}else {
    print("cant find room num")
}

~~~


옵셔널 체인닝을 토한 서브스크립트호출
옵셔널 벨류에있어서 바로 ? 를 통해 옵셔널을 걸 수 있다.

~~~
//옵셔널 체이닝을 통한 서브스크립트 호출
let optionalArray: [Int]? = [1, 2, 3]
//? 를통해 바로 옵셔널을 걸을 수 있다.
optionalArray?[1]

~~~