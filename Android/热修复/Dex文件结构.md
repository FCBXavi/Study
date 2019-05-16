Dex文件结构
======================
![dex文件图片](https://upload-images.jianshu.io/upload_images/1152636-8230c5995981b7c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/604/format/webp)         

是Android系统的可执行文件，包含应用程序的全部操作指令和运行时数据。java文件被编译成class文件后，使用dx工具将所有的class文件打包成一个dex文件，好处是其中各个类能够共享数据，降低冗余，使文件结构更加紧凑。

数据类型
-----------------------
1. u1 unit8_t,1字节无符号数       
2. u2 unit16_t,2字节无符号数     
3. u4 unit32_t,4字节无符号数        
4. u8 unit64_t,8字节无符号数      
5. sleb128 有符号LEB128,可变长度1～5      
6. uleb128 无符号LEB128       
7. uleb128p1 无符号LEB128值加1     
  
1~4表示1到8个字节的无符号数


dex文件结构
------------------------
1. header dex文件头部，记录整个dex文件的相关属性.             
2. string_ids 字符串数据索引，记录了每个字符串在数据区的偏移量.   
3. type_ids 类似数据索引，记录了每个类型的字符串索引.     
4. proto_ids 原型数据索引，记录了方法声明的字符串，返回类型字符串，参数列表.        
5. field_ids 字段数据索引，记录了所属类，类型以及方法名.       
6. method_ids 类方法索引，记录方法所属类名，方法声明以及方法名等信息.      
7. class_defs 类定义数据索引，记录指定类各类信息，包括接口，超类，类数据偏移量       
8. data 数据区，保存了各个类的真实数据               
9. link_data 连接数据区     

      
定义如下：      

		struct DexFile {
    		const DexHeader*    pHeader;
    		const DexStringId*  pStringIds;
    		const DexTypeId*    pTypeIds;
    		const DexFieldId*   pFieldIds;
    		const DexMethodId*  pMethodIds;
    		const DexProtoId*   pProtoIds;
    		const DexClassDef*  pClassDefs;
    		const DexLink*      pLinkData;
		}
		
Header
----------------------
记录dex文件的基本信息，长度固定是0x70，让虚拟机处理dex文件时不用考虑dex文件的多样性。          

| 字段名称 | 偏移值 | 长度 | 说明 |
| ------ | ------ | ------ | ------ |
|magic|0x0|8|魔数字段，值为"dex\n035\0"|
|checksum|0x8|4|校验码，校验头部是否损坏|
|signature|0xc|20|sha-1签名|
|file_size|0x20|4|dex文件总长度|
|header_size|0x24|4|文件头长度，009版本=0x5c,035版本=0x70|
|endian_tag|0x28|4|标示字节顺序的常量|
|link_size|0x2c|4|链接段的大小，如果为0就是静态链接|
|link_off|0x30|4|链接段的开始位置|
|map_off|0x34|4|map数据基址|
|string\_ids_size|0x38|4|字符串列表中字符串个数|
|string\_ids_off|0x3c|4|字符串列表基址|
|type\_ids_size|0x40|4|类列表里的类型个数|
|type\_ids_off|0x44|4|类列表基址|
|proto\_ids_size|0x48|4|原型列表里面的原型个数|
|proto\_ids_off|0x4c|4|原型列表基址|
|field\_ids_size|0x50|4|字段个数|
|field\_ids_off|0x54|4|字段列表基址|
|method\_ids_size|0x58|4|方法个数|
|method\_ids_off|0x5c|4|方法列表基址|
|class\_defs_size|0x60|4|类定义标中类的个数|
|class\_defs_off|0x64|4|类定义列表基址|
|data_size|0x68|4|数据段的大小，必须4k对齐|
|data_off|0x6c|4|数据端基址|


定义如下：  
   
	struct DexHeader {
    	u1  magic[8];           /* includes version number */
    	u4  checksum;           /* adler32 checksum */
    	u1  signature[kSHA1DigestLen]; /* SHA-1 hash */
    	u4  fileSize;           /* length of entire file */
    	u4  headerSize;         /* offset to start of next section */
    	u4  endianTag;
    	u4  linkSize;
    	u4  linkOff;
    	u4  mapOff;
    	u4  stringIdsSize;
    	u4  stringIdsOff;
    	u4  typeIdsSize;
    	u4  typeIdsOff;
    	u4  protoIdsSize;
    	u4  protoIdsOff;
    	u4  fieldIdsSize;
    	u4  fieldIdsOff;
    	u4  methodIdsSize;
    	u4  methodIdsOff;
    	u4  classDefsSize;
    	u4  classDefsOff;
    	u4  dataSize;
    	u4  dataOff;
	};          
		          
dex文件结构分析
-----------------
dalvik解析dex文件的内容，最终映射成DexMapList数据结构，DexHeader中的mapOff字段指定了DexMapList结构在dex在文件中的偏移。DexMapList声明如下。

	//位于/dalvik/libdex/DexFile.h
	struct DexMapList {
    	u4  size;               /* 个数 */
    	DexMapItem list[1];     /* DexMapItem的结构 */
	};	
size表示接下来有多少个DexMapItem结构

	struct DexMapItem {
    	u2 type;              /* kDexType开头的类型 */
    	u2 unused;            /*未使用，用于字节对齐 */
    	u4 size;              /* 类型的个数 */
    	u4 offset;            /* 类型的文件偏移 */
	};    
type是一个枚举常量，可以通过类型名称来判断DexMapItem类型     
	
	enum {
    	kDexTypeHeaderItem               = 0x0,
    	kDexTypeStringIdItem             = 0x1,
    	kDexTypeTypeIdItem               = 0x2,
    	kDexTypeProtoIdItem              = 0x3,
    	kDexTypeFieldIdItem              = 0x4,
    	kDexTypeMethodIdItem             = 0x5,
    	kDexTypeClassDefItem             = 0x6,
    	kDexTypeMapList                  = 0x0,
    	kDexTypeTypeList                 = 0x1,
    	kDexTypeAnnotationSetRefList     = 0x2,
    	kDexTypeAnnotationSetItem        = 0x3,
    	kDexTypeClassDataItem            = 0x0,
    	kDexTypeCodeItem                 = 0x1,
    	kDexTypeStringDataItem           = 0x2,
    	kDexTypeDebugInfoItem            = 0x3,
    	kDexTypeAnnotationItem           = 0x4,
    	kDexTypeEncodedArrayItem         = 0x5,
    	kDexTypeAnnotationsDirectoryItem = 0x6,
	};	
	
	

StringIdsOff区
----------------------
	
Header中string\_ids_off指的是字符串索引的位置，在这个区域，每四个字节指的是真正存放字符串的地址，这个地址在data区，每一个索引指向的真正字符串的位置，使用了MUTF-8编码，头部存放了由uleb128编码字符的个数，后面是真正的字符串内容。