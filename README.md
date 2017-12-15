# aliyun-java-sdk-dysmsapi

阿里云短信接口sdk

见阿里云地址 https://help.aliyun.com/document_detail/55359.html?spm=5176.doc55284.6.581.m3Ro21

由于阿里云不提供业务性质的maven中心库包，因此我将其源码上传到了我个人的maven中心库下。

maven 中心库地址

```
		<dependency>
			<groupId>com.github.1991wangliang</groupId>
			<artifactId>aliyun-java-sdk-dysmsapi</artifactId>
			<version>1.0.0</version>
		</dependency>

```


## 使用demo（springboot）

1. 配置文件（application.properties）下：

```


aliyun.msg.accessKeyId = xxx
aliyun.msg.accessKeySecret =  xxx
aliyun.msg.templateCode=  xxxx
aliyun.msg.signName=  xxxx


```

2. 定义接口：

```

public interface AliyunMsgService {

    boolean sendMsg(String mobile,String code) throws ServiceException;
}


```

3. 实现如下:

```


import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.dysmsapi.model.v20170525.SendSmsRequest;
import com.aliyuncs.dysmsapi.model.v20170525.SendSmsResponse;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;
import com.lorne.core.framework.exception.ServiceException;
import com.lorne.power.service.AliyunMsgService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

/**
 * create by lorne on 2017/12/4
 */
@Service
public class AliyunMsgServiceImpl implements AliyunMsgService {



    private Logger logger = LoggerFactory.getLogger(AliyunMsgServiceImpl.class);

    //产品名称:云通信短信API产品,开发者无需替换
    private static final String product = "Dysmsapi";
    //产品域名,开发者无需替换
    private static final String domain = "dysmsapi.aliyuncs.com";

    @Value("${aliyun.msg.accessKeyId}")
    private String accessKeyId;

    @Value("${aliyun.msg.accessKeySecret}")
    private String accessKeySecret;

    @Value("${aliyun.msg.templateCode}")
    private String templateCode;

    @Value("${aliyun.msg.signName}")
    private String signName;



    public boolean sendMsg(String mobile,String code) throws ServiceException{
        try {
            //可自助调整超时时间
            System.setProperty("sun.net.client.defaultConnectTimeout", "10000");
            System.setProperty("sun.net.client.defaultReadTimeout", "10000");

            //初始化acsClient,暂不支持region化
            IClientProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId, accessKeySecret);
            DefaultProfile.addEndpoint("cn-hangzhou", "cn-hangzhou", product, domain);
            IAcsClient acsClient = new DefaultAcsClient(profile);

            //组装请求对象-具体描述见控制台-文档部分内容
            SendSmsRequest request = new SendSmsRequest();
            //必填:待发送手机号
            request.setPhoneNumbers(mobile);
            //必填:短信签名-可在短信控制台中找到
            request.setSignName(signName);
            //必填:短信模板-可在短信控制台中找到
            request.setTemplateCode(templateCode);
            //可选:模板中的变量替换JSON串,如模板内容为"亲爱的${name},您的验证码为${code}"时,此处的值为
            request.setTemplateParam("{\"code\":\""+code+"\"}");

            //选填-上行短信扩展码(无特殊需求用户请忽略此字段)
            //request.setSmsUpExtendCode("90997");

            //可选:outId为提供给业务方扩展字段,最终在短信回执消息中将此值带回给调用者
            //request.setOutId("yourOutId");

            //hint 此处可能会抛出异常，注意catch
            SendSmsResponse response = acsClient.getAcsResponse(request);

            logger.info("msg-send->"+mobile+",res->"+response);

            System.out.println("短信接口返回的数据----------------");
            System.out.println("Code=" + response.getCode());
            System.out.println("Message=" + response.getMessage());
            System.out.println("RequestId=" + response.getRequestId());
            System.out.println("BizId=" + response.getBizId());

            if(response.getCode() != null && response.getCode().equals("OK")) {
                //请求成功
                return true;
            }
            return false;
        }catch (Exception e){
            throw new ServiceException(e);
        }
    }
}


```
