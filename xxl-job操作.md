# xxl-job

## 1.xxl:

  job:
    admin:
      addresses: http://ykd-xxl:9080/xxl-job-admin  # xxl-job-admin 接口地址
    executor:
      port: 9988   #通讯端口
      appName: test-xxl

## 2.启动类添加注解

@EnablePigxXxlJob

## 3.创建数据库   

xxl-job

## 4.修改xxl-job-admin   

application.properties

## 5.修改xxl-job-admin   

logback.xml

## 6.xxl-job-admin 进行打包   

不执行测试用例，也不编译测试用例类。

 mvn clean install -Dmaven.test.skip=true

## 7.运行jar包  target目录下

java -jar xxl-job-admin-2.2.0.jar  --server.port = 9080

linux：**nohup java -jar  XXXX.jar  &**

## 8.访问中心

localhost:9080/xxl-job-admin

## 9.执行器

@Slf4j
@Component
@JobHandler("xxxHandler")
public class xxxHandler extends IJobHandler{

}

xxlJobUtil

```java

/**
 * @author liguo
 * @Description
 * @date 2020/11/6/006 16:49
 */
@Slf4j
@Component
public class XxlJobUtil {
	@Value("${xxl.job.admin.addresses}")
	private String adminAddresses;

	@Value("${xxl.job.executor.appName}")
	private String appname;

	private RestTemplate restTemplate = new RestTemplate();

	private static final String ADD_URL = "/jobinfo/addJob";
	private static final String UPDATE_URL = "/jobinfo/updateJob";
	private static final String REMOVE_URL = "/jobinfo/removeJob";
	private static final String PAUSE_URL = "/jobinfo/pauseJob";
	private static final String START_URL = "/jobinfo/startJob";
	private static final String ADD_START_URL = "/jobinfo/addAndStart";
	private static final String GET_GROUP_ID = "/jobgroup/getGroupId";


	public String add(XxlJobInfo jobInfo){
		// 查询对应groupId:
		Map<String,Object> param = new HashMap<>();
		param.put("appname", appname);
		String json = JSON.toJSONString(param);
		String result = doPost(adminAddresses + GET_GROUP_ID, json);

		JSONObject jsonObject = JSON.parseObject(result);
		String groupId = jsonObject.getString("content");
		jobInfo.setJobGroup(Integer.parseInt(groupId));
		String json2 = JSON.toJSONString(jobInfo);
		return doPost(adminAddresses + ADD_URL, json2);
	}


	public String update(int id, String desc, String executorHandler, String executorParam, String cron){
		Map<String,Object> groupParam = new HashMap<>();
		groupParam.put("appname", appname);
		String groupJson = JSON.toJSONString(groupParam);
		String result = doPost(adminAddresses + GET_GROUP_ID, groupJson);
		JSONObject jsonObject = JSON.parseObject(result);
		String groupId = jsonObject.getString("content");
		Map<String,Object> param = new HashMap<>();
		param.put("id", id);
		param.put("jobGroup",Integer.parseInt(groupId));
		param.put("jobDesc",desc);
		param.put("author","ykd");
		param.put("executorRouteStrategy","ROUND");
		param.put("executorBlockStrategy","SERIAL_EXECUTION");
		param.put("jobCron", cron);
		param.put("executorHandler",executorHandler);
		param.put("executorParam", executorParam);
		param.put("executorTimeout",0);
		param.put("executorFailRetryCount",0);
		param.put("triggerNextTime", 0);
		String json = JSON.toJSONString(param);
		return doPost(adminAddresses + UPDATE_URL, json);
	}

	public String remove(int id){
		Map<String,Object> param = new HashMap<>();
		param.put("id", id);
		String json = JSON.toJSONString(param);
		return doPost(adminAddresses + REMOVE_URL, json);
	}

	public String pause(int id){
		Map<String,Object> param = new HashMap<>();
		param.put("id", id);
		String json = JSON.toJSONString(param);
		return doPost(adminAddresses + PAUSE_URL, json);
	}

	public String start(int id){
		Map<String,Object> param = new HashMap<>();
		param.put("id", id);
		String json = JSON.toJSONString(param);
		return doPost(adminAddresses + START_URL, json);
	}

	public String addAndStart(XxlJobInfo jobInfo){
		Map<String,Object> param = new HashMap<>();
		param.put("appname", appname);
		String json = JSON.toJSONString(param);
		String result = doPost(adminAddresses + GET_GROUP_ID, json);

		JSONObject jsonObject = JSON.parseObject(result);
		String groupId = jsonObject.getString("content");
		jobInfo.setJobGroup(Integer.parseInt(groupId));
		String json2 = JSON.toJSONString(jobInfo);

		return doPost(adminAddresses + ADD_START_URL, json2);
	}

	public String doPost(String url, String json){
		HttpHeaders headers = new HttpHeaders();
		headers.setContentType(MediaType.APPLICATION_JSON);
		HttpEntity<String> entity = new HttpEntity<>(json ,headers);
		log.info(entity.toString());
		ResponseEntity<String> stringResponseEntity = restTemplate.postForEntity(url, entity, String.class);
		return stringResponseEntity.getBody();
	}
}

```





