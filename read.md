##  Simple Smart-Contract สำหรับตรวจสอบสินค้าว่าเป็นสินค้าจริงหรือปลอมแบบ
เนื่องจากปัจจุบันมีสินค้าละเมิดลิขสิทธิ์อยู่มากอีกทั้งยังมีธุรกิจใหม่ๆเกิดขึ้นเช่นการขายสินค้ามือสอง หรือ การซื้อสินค้าใหม่มา resell ในกรณีสินค้าที่มีลักษณะเฉพาะเป็นสินค้ารุ่นหายาก
ซึ่ง มักมีการทำสินค้าเลียนแบบ ซึ่งปัญหานี้ทำให้เกิดแนวคิดในการใช้เทคโนโลยี Blockchain ของ Ethereum ในการแก้ปัญหาว่าสินค้าเหล่านั้นเป็นสินค้าจริงหรือปลอม หากผู้ผลิตได้มี   การบันทึก Serial Number ลงบน Blockchain แล้วนั้น เราก็จะสามารถนำ Serial Number ของสินค้าที่จซื้อมาตรวจสอบว่าสินค้านั้นสินค้าจริงหรือปลอมได้
### การใช้งาน
นำ source code ไป compile และ deploy โดยใช้ Remix IDE  

**_code ชุดนี้ไม่ได้มีการปกป้องให้ user อื่นๆทำการ addProduct เองหากต้องการนำไปทดลองสำหรับโปรเจคควร_**  
**_เพิ่ม modifier สำหรับ control function ให้ทางผู้ผลิตที่ถูกแต่งตั้งแล้วเท่านั้นที่สามารถ addProduct ได้_**

```
    
        mapping(address => bool) owner;
        mapping(address => bool) authorized;
        
        event AddedAuthorized(address addAuthorized);
        event RemovedAuthorized(address removeAuthorized);
        
        constructor () public {
          owner[msg.sender] = true;
        }

        modifier onlyOwner() {
          require(owner[msg.sender]);
          _;
        }
    
        modifier onlyAuthorized() {
          require(authorized[msg.sender]);
          _;
        }
        
        function grantAuthorized(address _toAdd) public onlyOwner {
           require(_toAdd != address(0));
           authorized[_toAdd] = true;
           emit AddedAuthorized(_toAdd); 
        }

        function revokeAuthorized(address _toRemove) public onlyOwner {
          require(_toRemove != address(0));
          require(_toRemove != msg.sender);
          authorized[_toRemove] = false;
          emit RemovedAuthorized(_toRemove); 
        }

```
**_เ่พิ่ม modifier onlyOwner() คือผู้สร้าง contract ซึ่งมีสิทธ์ grantAuthorized หรือ revokeAuthorized ผู้ผลิตเพื่อใช้ function addProduct_**  

```
//Example of addProduct with modifier
    function addProduct (Product memory _products) public onlyAuthorized {
       bytes32 hash = keccak256(abi.encode(_products.serialnumber));
       products[hash] = _products;
    }
```

ใช้ function addManyProducts สำหรับการเพิ่มสินค้าจริงครั้งละเยอะ ๆ
       ตัวอย่าง input
```
[["nike","nk001","Air Max 97"],
 ["nike","nk002","Free Flykinit"],
 ["addidas","add001","ULTRABOOST 19"],
 ["addidas","add002","NITE JOGGER"],
 ["asics","as001","GEL-Nimbus 21"],
 ["asics","as002","ASICS GEL-Nimbus 21"]]
```

ใช้ function checkProduct สำหรับการรวจสอบสินค้าจริงหรือปลอม  
ตัวอย่างโดยกาใส่ input เป็น Serial Number ของสินค้าลงไป  
- Input  
```
 nk001
 //ผลลัพธ์ที่คาดหวัง คือ ["nike","nk001","Air Max 97"]
 //หาก output return ค่าว่างนั้นหมายความว่าเป็นสินค่าปลอม
```
- Output 
```
 0: string: brand nike
 1: string: describe Air Max 97
 2: string: serialnumber nk001
// หมายความว่า nk001 เป็นสินค้าแท้ไม่ใช่สินค้าของปลอม
```

