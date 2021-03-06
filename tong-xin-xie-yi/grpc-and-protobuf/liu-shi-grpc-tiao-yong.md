# 流式grpc调用

## 流式grpc接口定义\(pb\)

```text
service FileService {
    //file manager 通过http上传到cdn, 原python GarenaFileManager的upload
    rpc UploadFile(stream UploadFileReq) returns (UploadFileRsp);
    ...
}
```

## 接口调用\(Python\) -- 迭代器

参考代码

```text
class FileManage(TestCase):
    '''demo
    '''
    owner = "enbo.wang"
    status = TestCase.EnumStatus.Ready
    priority = TestCase.EnumPriority.Normal
    timeout = 1

    def pre_test(self):
        super(FileManage, self).pre_test()
        
    #迭代器
    def datagenerator(self):
        print("start")
        with open(r'/Users/enbowang/Documents/my_code/AirTestFrame/airtestproj/resources/Airpay_File_Service/test_pic_2.png','rb') as f:
            line = f.readline()
            while line:
                #print(base64.b64encode(line))
                h = hashlib.md5()
                h.update(line)
                #print(h.hexdigest())
                data = file_service_pb2.UploadFileReq(
                    data=line,
                    check_sum=h.hexdigest()
                )
                yield data
                line = f.readline()
        print("stop")

    def run_test(self):
        self._channel = grpc.insecure_channel("%s:%d" % (risk_svr_ip, risk_svr_port))
        self.stub = grpcstub.GrpcStub.get_client(FileServiceStub)(self._channel)

        #接口调用参数使用迭代器
        self.start_step("接口调用....")
        self.a = self.datagenerator()
        res, err = self.stub.UploadFile(self.a)
        print(res,err)

        self.start_step("数据校验....")

        self.start_step("链接释放....")
        self._channel.close()
```

 

