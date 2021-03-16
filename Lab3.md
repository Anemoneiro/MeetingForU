# Lab3实验报告

项目名称：MeetingForU

项目描述：一个能够进行会议投稿及管理的新型会议系统。



## 页面需求规划截图

![](imgs/demand-planning1.png)

![](imgs/demand-planning2.png)

![](imgs/demand-planning3.png)

![](imgs/demand-planning4.png)

![](imgs/demand-planning5.png)

![](imgs/demand-planning6.png)

![](imgs/demand-planning7.png)

![](imgs/demand-planning8.png)

![](imgs/demand-planning9.png)



## 项目功能流程图


![](imgs/workline.png)



## 项目代码检查截图

#### 后端代码检查截图


![](imgs/backend-codecheck.png)



## 项目tag截图



#### 后端tag截图

由于最早的tag截图丢失，后端在此对此前一次tag缺失提交打tag，并展示所有tag

![](imgs/gittag-backend.png)



## 项目各页面截图及使用说明

登陆界面
![](imgs/MeetingForU/login.png)

注册界面
![](imgs/MeetingForU/register.png)

用户页面 - 会议申请界面
![](imgs/MeetingForU/confRegi.png)

用户页面 - 我的会议申请界面
![](imgs/MeetingForU/myApplication.png)

用户页面 - 我的会议界面
![](imgs/MeetingForU/myConf.png)

管理员页面 - 会议申请处理
![](imgs/MeetingForU/adminpage.png)

Chair页面 - 会议信息
![](imgs/MeetingForU/chairpage.png)

Chair页面 - 搜索用户
![](imgs/MeetingForU/chairSearchPC.png)

Chair页面 - 搜索结果展示
![](imgs/MeetingForU/chairSelectResult.png)

Chair页面 - 显示所有PC member
![](imgs/MeetingForU/chairSelectPC.png)

Chair页面 - 已发送PC member邀请
![](imgs/MeetingForU/chairCheckInvitations.png)

用户页面 - 收到PC member邀请 
![](imgs/MeetingForU/myReceivedInvitation.png)

用户页面 - 所有会议信息
![](imgs/MeetingForU/allConfs.png)

用户页面 - 投稿页面
![](imgs/MeetingForU/contribute.png)

Author - 我的投稿
![](imgs/MeetingForU/myContribution.png)



## 实验过程记录

#### 第一次会议对功能进行分析 2020/4/3 By段欣然

##### 用户注销

用户在登录系统之后，可以进行用户的注销，在用户点击注销按钮的时候，需要让用户对其注销操作进行确认（是否确定注销用户）。在用户成功进行账号的注销之后系统自动跳转到登录页面。

前端，无交互需要；没有特定页面，有弹窗。
前端注销用户，在localhost中删除token即可。

##### 会议审核

管理员对于会议申请进行审核，可以通过会议申请或者驳回会议申请，当会议申请通过之后，申请人自动成为该会议的chair。申请者可以在系统内查看自己提交的申请的状态，比如说审核中，已通过和未通过。此次实验中，管理员只需要实现这⼀个功能。

前端+后端，有交互需要；有特定页面，有弹窗。
管理员页面：
前端页面基于会议，显示一个列有所有待审核会议信息表单，每个会议旁有通过和拒绝的按钮，点击按钮应该有确认弹窗，再次点击确认之后才会进行通过或者驳回。
后端实现将数据库内待审核会议信息传输给前端并显示。
这里的管理员应该是系统管理员，后端需要对管理员页面进行权限验证。
申请人（用户）页面：
前端页面基于用户个人申请，显示一个列有该用户所有申请的信息表单，没有按钮，只有三种审核状态。
后端实现数据库内个人审核信息传输给前端并显示。
这里的难点在于数据库的绑定。

##### 会议及角色选择

在用户登录系统之后，如果想要对会议事务进行处理，就首先要确定自己所要处理的会议以及以什么身份进入该会议。首先系统应该列出该用户参与的所有会议，在用户选择完会议之后，系统应列出在该用户在该会议中的所有角色。只有这两个选项都选择完成之后，才能对其角色相关的事务进行处理。

前端+后端，有交互需要；有特定页面。
前端页面基于用户参与会议，显示一个列有该用户所有参与会议的信息表单，点击会议后课跳转至会议的具体页面，在具体页面中显示该会议的详细信息，以及该用户在该会议中的所有角色。
后端实现数据库内个人会议信息传输给前端并显示。这里的会议一定要是审核通过的。

##### 邀请会议的PC member

当用户以chair的身份进入某个会议之后，就可以邀请系统用户成为该会议的PC member。chair按照用户姓名进行查询，查出所有相关的用户（有可能有重名的用户）以及用户的详细信息，如邮箱，区域等。chair勾选⼀个或多个用户发送邀请。chair还可以对自己发送邀请的状态进行查询，邀请状态可以是待确认，已同意或者已拒绝。

前端+后端，有交互需要；无特定页面。
Chair页面：
前端页面由会议选择页面扩展，当用户角色为chair时，显示一个搜索框和搜索按钮，输入信息后点击按钮，按钮下面会出现一个表单，显示所有符合要求的用户及其详细信息。用户前面有多选框，勾选后点击提交按钮即可发送邀请至后端仓库。查询规定全名以这个字段为开头就可以被查询出来。
后端需要对Chair页面进行权限验证，实现一个基于会议信息的邀请仓库，向前端发送用户信息，交互接受chair的邀请信息存放在仓库中。

##### 接受邀请并成为PC member

用户可以在会议邀请通知界面查看所有未处理的PC member邀请， 系统显示的邀请的具体信息包括chair的姓名，会议的名称等。对于每个邀请用户可以同意或者拒绝，如果同意邀请，则会立即成为该会议的PC member。

前端+后端，有交互需要；有特定页面，有弹窗。
前端页面基于会议邀请，存在一个表单显示所有未处理的邀请信息，每个邀请信息有同意和拒绝两个按钮，点击按钮应该有确认弹窗，再次点击确认之后才会进行同意或者拒绝。
后端需要从前端接受确认信息并对邀请信息仓库进行操作，将请求从仓库中移除并修改用户信息/会议信息。

##### 开启投稿

当用户以chair身份进入某个会议，并且该会议的状态是审核通过时，用户可以开启该会议的投稿。当该会议投稿开启之后，其他系统用户才能在该会议中进行投稿。

前端+后端，有交互需要；无特定页面。
Chair页面：
前端页面由会议选择页面扩展，当用户角色为chair，并且会议状态审核通过时，显示一个开启投稿按钮，点击按钮，点击按钮应该有确认弹窗，再次点击确认之后才会进行投稿开启。
后端需要对Chair页面进行权限验证，接受前端信息，更改投稿接受状态。

##### 论文投稿

在用户登录进入系统之后，除了可以以某一具体的角色进入到某个具体的会议处理事务之外，还可以直接查看当前所有的会议列表（列表中显示的会议⼀定是通过审核的）。所有的会议还需要显示其状态 （审核通过和投稿中）。选中一个会议，如果该用户不是该会议的 chair，则可以直接进行投稿，投稿的时候需要填写论文的标题（长度限制50个字符），摘要（长度限制800个字符），还需要进行论文的文件上传（只要pdf文件），当前上传者就是论文作者。

前端+后端，有交互需要；有特定页面。
用户页面：
前端有两个页面。
前端第一个页面由会议及角色选择页面扩展，显示一个表单，表单里是该用户参与的已通过审核的所有会议信息，点击该会议，如果是chair则弹窗不能投稿，不是chair直接跳转至投稿页面，投稿页面即上传表单。
后端需要对页面进行权限验证，返回该用户参与的所有已审核的会议到已审核会议表单中，接受前端上传的表单，需要在数据库内实现文件的存储。

补充：共有chair，PC member和author三种身份，PC member自己也可以投稿，但是不允许审自己的稿。
1）审稿的规则暂时不用考虑，本次lab不涉及
2）一个会议的chair不能投稿也不需要再成为PC member
3）一个会议的PC member或任何用户都可以对这个会议投稿，投稿后成为author身份
4）用户如果是一个会议的author那么也可以选择此身份进入会议，作为author可以查看在会议中的论文投稿情况
投稿的时候可以没有任何身份，选择一个会议即可投稿（会议的状态等要符合要求，lab需求中有说明）
投稿成功后就成了相应会议的author，就可以点击这个会议的author角色进入作为作者查看自己的投稿



#### 后端代码最终测试结果截图


![](imgs/backend-testimg.jpg)



#### 开发过程中存在的问题总结



##### 问题描述

测试写在lab2applicationTest.java内，未登录直接进行操作，导致用户没有token从而不能进行测试。

##### 解决方法

将测试改为单元测试，每次测试都选择先登录，然后再进行需要登录权限的操作。

```java
String username = "zhangsan";
String password = "wobushizs123";
authService.login(username, password);
```

##### 问题描述

后端postman测试运行正确代码在其他组员主机上无法运行。

##### 解决方法

通过检查，发现后端同学push代码之前没有使用git add .，导致分支中的新建内容没有push到华为云，其他组员pull的代码出现缺失，及时add并pull即可解决问题。

##### 问题描述

后端Meeting类中的MeetingAuthority与User类无法进行对应。

##### 解决方法

创建了一个新的实体类：UserMeetingAuthPair类，这个实体类除了id元素，还有三个字段：username、meetingfullname、meetingAuthority，这三个字段的联合查询可保证结果唯一，而依赖单个字段查询可以返回与单个用户/单个会议相关的所有权限信息。

##### 问题描述

交互时发现了即会议注册传入的时间会比测试时注册的时间少8个小时。

##### 解决方法

发现时间差异是时区不同而导致的，因此在每个Date对象里添加如下代码：

```java
@DateTimeFormat(pattern = "yyyy-MM-dd")
@JsonFormat(pattern = "yyyy-MM-dd", timezone = "GMT+8")
private Date holdtimestart;
```

最后在配置文件中添加如下代码：

```java
# JACKSON (JacksonProperties)
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
```

##### 问题描述

前端在远程主机部署后向后端发送请求无效。

##### 解决方法

通过观察前端传输路径，发现前端传入/api/register，而后端接收端口为/register，在后端application.properties文件中作如下修改即可。

```java
server.servlet.context-path=/api
```



## 个人实验总结



#### 段欣然 18307130295



##### 实验任务

这次实验中，我主要负责后端的交互部分以及部分功能的实现，其中包括对后端源代码打标签，编写后端过滤器的代码，编写管理员页面功能代码，编写按照会议申请状态、申请人名称查询会议的功能代码，编写返回用户参与的会议及用户权限功能代码，编写管理用户会议权限的中间表及其相关查询功能代码，编写会议邀请类及用户同意/拒绝成为PC Member的功能代码，编写开启会议投稿功能代码，编写返回用户投稿相关代码，并编写所有与自己的任务相关单元测试代码，使用postman进行测试与debug，规定接口相关信息编写所有接口功能代码。除此之外，我还在第一次会议之中进行以任务为单位分析功能并整理成文档，对于需求规划面板进行整理，部署后端项目，以及部署完毕后进行交互与功能测试。



##### 实验部分核心功能代码一览

首先，第一步是修改原先的安全设置，保证登录过滤器和管理员过滤器的正常运行。

```java
SecurityConfig.java
/*对于安全设置进行修改，阻止普通用户访问admin页面，对于用户在会议中的权限通过调用方法时的exception处理*/
    
    ...
                .authorizeRequests()
                .antMatchers("/welcome","/login","/register").permitAll()
                .antMatchers("/admin/**").hasAuthority("Admin")
                .anyRequest().authenticated();
    ...
```

接下来的一个重点是对于交互接口的规定及代码编写，它的编写要随时与前后端开发进度同步，以保证可以随时进行功能测试。

```java
ConfController.java
/*由于交互部分代码基本相同，在会议相关Controller中仅提供部分全体展示*/

	/*返回用户参与的所有会议*/
    @PostMapping("/myMeeting")
    public ResponseEntity<?> myMeeting() {
        logger.debug("get myMeeting");
        Map<String, Object> response = new HashMap<>();
        List<Meeting> myMeetings = confService.getAllMyMeeting();
        System.out.println(myMeetings.size());
        response.put("size",myMeetings.size());
        response.put("myMeetings",myMeetings);
        return ResponseEntity.ok(response);
    }

	/*用户对Chair发送的成为PC member的邀请进行处理*/
    @PostMapping("/user/dealWithInvitation")
    public ResponseEntity<?> UserDealWithInvitation(@RequestBody UserDealWithInvitationRequest request) {
        logger.debug("DealWithReceivedInvitation:" + request.toString());
        Map<String, Object> response = new HashMap<>();
        confService.UserDealWithInvitation(request.getMeetingFullname(), request.getSendingDate(),
                request.getInvitationStatus());
        response.put("status","Success");
        return ResponseEntity.ok(response);
    }

	/*Chair功能：通过全名搜索用户*/
    @PostMapping("/chair/searchUser")
    public ResponseEntity<?> chairWorkSpaceSearchUser(@RequestBody MeetingChairSearchUserRequest request) {
        logger.debug("admin get PC");
        System.out.println(request.getMeetingFullname());
        System.out.println(request.getFullnamePart());
        Map<String, Object> response = new HashMap<>();
        List<User> userList= confService.getAllUserByUsernamePart(request.getMeetingFullname(), request.getFullnamePart());
        response.put("userList", userList);
        return ResponseEntity.ok(response);
    }
```

需要注意的是，最后对于会议投稿的控制器与其他控制器的实现有些不同，需要特殊编写。

```java
PaperController.java
/*投稿状态由于前端请求存在文件，Context-type为multipart/form-data，因此后端不能使用@RequestParam来接收请求*/

	@PostMapping("/user/contribute")
    public ResponseEntity<?> contributeFile(@RequestParam("title")String title, @RequestParam("summary")String summary,
                   @RequestParam("file") MultipartFile file, @RequestParam("meetingFullname")String meetingFullname) {
        logger.debug("contributeFile");
        Map<String, Object> response = new HashMap<>();
        paperService.upload(file,title, summary, meetingFullname);
        response.put("status","Success");
        return ResponseEntity.ok(response);
    }
```

本次所编写的所有后端交互Controller及相应Request类如下所示：

```java
    @PostMapping("/admin/stuff")
    public ResponseEntity<?> meetingToManage()

    @PostMapping("/admin/workspace")
    public ResponseEntity<?> adminChangeMeetingStatus(@RequestBody MeetingStatusChangeRequest request)

    @PostMapping("/myMeetingApplication")
    public ResponseEntity<?> myMeetingApplication()

    @PostMapping("/myMeeting")
    public ResponseEntity<?> myMeeting()

    @PostMapping("/myInvitation")
    public ResponseEntity<?> myReceivedInvitation()

    @PostMapping("/user/dealWithInvitation")
    public ResponseEntity<?> UserDealWithInvitation(@RequestBody UserDealWithInvitationRequest request)

    @PostMapping("/user/getAllMeeting")
    public ResponseEntity<?> userGetAllMeeting()
        
    @PostMapping("/chair/searchUser")
    public ResponseEntity<?> chairWorkSpaceSearchUser(@RequestBody MeetingChairSearchUserRequest request)
        
    @PostMapping("/chair/invitePC")
    public ResponseEntity<?> invitePC(@RequestBody MeetingInvitationRequest request)
        
    @PostMapping("/chair/invitation")
    public ResponseEntity<?> chairGetSentInvitations(@RequestBody ChairGetSentInvitationsRequest request)
        
    @PostMapping("/chair/getPC")
    public ResponseEntity<?> getPCOfMeeting(@RequestBody ChairGetMeetingAllPCmemberRequest request)
        
    @PostMapping("/chair/deletePC")
    public ResponseEntity<?> deletePCOfMeeting(@RequestBody ChairDeletePCmemberRequest request)
        
    @PostMapping("/chair/startContribute")
    public ResponseEntity<?> chairStartContribute(@RequestBody ChairStartContributeRequest request)
        
    @PostMapping("/user/contribute")
    public ResponseEntity<?> contributeFile(@RequestParam("title")String title, @RequestParam("summary")String summary,
                   @RequestParam("file") MultipartFile file, @RequestParam("meetingFullname")String meetingFullname)

    @PostMapping("/author/getContribution")
    public ResponseEntity<?> authorGetContribution(@RequestBody AuthorGetContributionRequest request)
```

对于Request类的设计，我们需要注意的是，Request类中与前端接口所统一的是get和set方法中的字段，默认第一个大写字母为小写，由于该类编写众多且格式相同，在此仅展示本人编写的其中一者作为样例。

```java
UserDealWithInvitationRequest.java
/*Request类编写的样例展示*/

public class UserDealWithInvitationRequest {
    private String meetingfullname;
    private String sendingDate;
    private String invitationStatus;

    public UserDealWithInvitationRequest() {}

    public UserDealWithInvitationRequest(String meetingfullname, String sendingDate, String invitationStatus){
        this.meetingfullname = meetingfullname;
        this.sendingDate = sendingDate;
        this.invitationStatus = invitationStatus;
    }

    public String getMeetingFullname() {
        return meetingfullname;
    }

    public void setMeetingFullname(String meetingfullname) {
        this.meetingfullname = meetingfullname;
    }

    public String getSendingDate() {
        return sendingDate;
    }

    public void setSendingDate(String sendingDate) {
        this.sendingDate = sendingDate;
    }

    public String getInvitationStatus() {
        return invitationStatus;
    }

    public void setInvitationStatus(String invitationStatus) {
        this.invitationStatus = invitationStatus;
    }

}
```

对于单个Test类的设计，我们需要保证字段不重复，在测试正确的基础上不干扰其他Test类的实现。如果测试失败，需要通过断点输出法排查究竟是哪一个步骤出现了问题，并进行完善或改正。

```java
ConfServiceTest.java
/*测试类编写的样例展示*/

	@Test
    public void UserDealWithInvitation() {
        AuthService authService= new AuthService(userRepository, authorityRepository, authenticationManager,
                jwtUserDetailsService, jwtTokenUtil, passwordEncoder);
        ConfService confService = new ConfService(userRepository, authorityRepository, meetingRepository,
                meetingAuthorityRepository, jwtUserDetailsService, userMeetingAuthPairRepository, meetingInvitationRepository);

        RegisterRequest regi = new RegisterRequest("AAA5","wobushizs123","zhangBBBsan",
                "zs@126.com","China","Shanghai");
        authService.register(regi);

        String username = "admin";
        String password = "password";
        authService.login(username, password);

        MeetingRegisterRequest request1 = new MeetingRegisterRequest("ZPL",
                "Zhangsan's Personal legal Meeting8", "2020-01-13",
                "2020-01-16", "Xiahai", "2020-4-19", "2020-4-30");
        confService.meetingRegister(request1);

        MeetingStatusChangeRequest changeReq1 = new MeetingStatusChangeRequest(
                "Zhangsan's Personal legal Meeting8", "adminAgree");

        Meeting changedR1 = confService.meetingStatusChange(changeReq1.getMeetingFullname(),
                changeReq1.getMeetingStatus());

        List<String> receiverList = new ArrayList<>();
        receiverList.add("AAA5");

        confService.sendMeetingInvitation("Zhangsan's Personal legal Meeting8",
                "2020-01-01", receiverList);

        //System.out.println(meetingInvitationRepository.findAll());

        username = "AAA5";
        password = "wobushizs123";
        authService.login(username, password);

        MeetingInvitation invitation = meetingInvitationRepository
                .findBySendingDateAndMeetingfullnameAndReceiverUsernameAndInvitationStatus("2020-01-01",
                        "Zhangsan's Personal legal Meeting8", "AAA5",
                        "ToBeConfirmed");
        assertNotNull(invitation);

        confService.UserDealWithInvitation("Zhangsan's Personal legal Meeting8",
                "2020-01-01", "PCAgree");

        List<MeetingInvitation>invitations = meetingInvitationRepository.findByInvitationStatus("PCAgree");

        assertNotNull(meetingInvitationRepository
                .findBySendingDateAndMeetingfullnameAndReceiverUsernameAndInvitationStatus("2020-01-01",
                        "Zhangsan's Personal legal Meeting8", "AAA5",
                        "PCAgree"));

    }
```

为了解决用户在不同会议中对应不同身份信息的问题，我创建了一个新的实体类：UserMeetingAuthPair类，这个实体类除了id元素，还有三个字段：username、meetingfullname、meetingAuthority，这三个字段的联合查询可保证结果唯一，而依赖单个字段查询可以返回与单个用户/单个会议相关的所有权限信息。

```java
UserMeetingAuthPair.java
/*存储用户在不同会议中对应不同身份信息的实体类，username+meetingfullname+meetingAuthority三个字段联合查询，可保证结果唯一*/

@Entity
public class UserMeetingAuthPair {
    private static final long serialVersionUID = -131986782337490104L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String username;
    private String meetingfullname;

    @ManyToOne(cascade = CascadeType.MERGE, fetch = FetchType.EAGER)
    @JsonIgnore
    private MeetingAuthority meetingAuthority = new MeetingAuthority();


    public UserMeetingAuthPair(){}

    public UserMeetingAuthPair(String username, String meetingfullname) {
        this.username=username;
        this.meetingfullname = meetingfullname;
    }

    public UserMeetingAuthPair(String username, String meetingfullname,MeetingAuthority meetingAuthority) {
        this.username=username;
        this.meetingfullname = meetingfullname;
        this.meetingAuthority=meetingAuthority;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUsername(){
        return username;
    }

    public void setMeetingfullname(String meetingfullname) {
        this.meetingfullname = meetingfullname;
    }

    public String getMeetingfullname(){
        return meetingfullname;
    }

    public MeetingAuthority getMeetingAuthority() {
        return meetingAuthority;
    }

    public void setMeetingAuthority(MeetingAuthority meetingAuthority) {
        this.meetingAuthority = meetingAuthority;
    }

}
```

但是，引入这个实体类的一个缺陷是无法在SecurityConfig中进行过滤，于是我通过添加Exception类来解决这个问题：对于与权限相关的任务进行权限检查，在进行操作之前取出UserMeetingAuthPair进行判断，如果进行操作的用户没有相关权限则抛出异常，过程终止。下面以对于需要Chair权限的功能的异常控制为例进行展示：

```java
NotHaveChairAuthorityException.java
/*在用户没有Chair权限时抛出异常*/

public class NotHaveChairAuthorityException extends RuntimeException {
    private static final long serialVersionUID = -2553223088442141676L;

    public NotHaveChairAuthorityException(String username, String meetingFullname) {
        super("User '" + username + "' don't have chair authority in meeting '" + meetingFullname + "'.");
    }
}


ControllerAdvisor.java
/*对于异常进行处理*/
    @ExceptionHandler(NotHaveChairAuthorityException.class)
    ResponseEntity<?> handleNotHaveChairAuthorityException(NotHaveChairAuthorityException ex) {
        Map<String, String> response = new HashMap<>();
        response.put("message", ex.getMessage());
        return new ResponseEntity<>(response, HttpStatus.FORBIDDEN);
    }


ConfService.java
/*异常类实装展示*/
...
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        User sender = userRepository.findByUsername(username);
        MeetingAuthority ChairAuth = meetingAuthorityRepository.findByMeetingAuthority("Chair");
        if(null == userMeetingAuthPairRepository.findByUsernameAndMeetingfullnameAndMeetingAuthority(username,
                meetingfullname, ChairAuth)){
            throw new NotHaveChairAuthorityException(username, meetingfullname);
        }

...
```

此外，我所实现的所有功能类如下所示：

```java
    //Xinran Duan's work
    public List<Meeting> getMyMeetingApplication()
    public List<Meeting> getAllToBeAppliedMeeting()
    public List<Meeting> getAllMeeting()
    public List<Meeting> getAllMyMeeting()
    public Meeting meetingStatusChange(String meetingFullname, String meetingStatus)
    public List<User> getAllUserByUsernamePart(String meetingfullname, String usernamePart)
    public void sendMeetingInvitation (String meetingfullname, String sendingDate, List<String> receiverList)
    public List<MeetingInvitation> chairGetSentInvitations(String meetingfullname)
    public void UserDealWithInvitation(String meetingfullname, String sendingDate, String invitationStatus)
    public void chairStartContribute(String meetingfullname)
    public List<Paper> authorGetContribution(String meetingFullname)
```

下面我将对我所实现的几个方法进行具体展示：

```java
ConfController.java
/*返回用户所参与的所有会议*/

    public List<Meeting> getAllMyMeeting() {
    	//从SecurityContextHolder中取得用户名，初始化相关变量
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        User user = userRepository.findByUsername(username);
    	//返回的会议信息，还没有注入用户的权限信息
        Set<Meeting> meetingSet = user.getMeetings();
    	//作为return的会议信息，需要注入用户的权限信息
        List<Meeting> meetingRtn = new ArrayList<Meeting>();
        Iterator<Meeting> iterator = meetingSet.iterator();
        Meeting realMeeting;
        Meeting tmp;
        UserMeetingAuthPair tmpPairs;

        while(iterator.hasNext()) {
            realMeeting = iterator.next();
            tmp = meetingRepository.findByMeetingfullname(realMeeting.getMeetingFullname());
            String meetingFullname = tmp.getMeetingFullname();
            //在仓库中查询用户在此会议中的所有权限
            List<UserMeetingAuthPair> userMeetingAuthPairs = userMeetingAuthPairRepository.
                    findByUsernameAndMeetingfullname(username, meetingFullname);

            Iterator<UserMeetingAuthPair> iteratorofPairs = userMeetingAuthPairs.iterator();
            Set<MeetingAuthority> meetingAuth = new HashSet<MeetingAuthority>();
            while(iteratorofPairs.hasNext()) {
                tmpPairs = iteratorofPairs.next();
                meetingAuth.add(tmpPairs.getMeetingAuthority());
                //在用户的meetingAuth集合中注入用户在此会议中的所有权限
            }
            //将权限设置在当前会议上，并加入返回队列，注意此处的修改没有保存在仓库，Meeting对象中的权限集合相当于临时变量
            tmp.setMeetingAuthorities(meetingAuth);
            meetingRtn.add(tmp);
        }

        return meetingRtn;
    }
```

```java
ConfController.java
/*管理员审批会议，改变会议状态*/

	public Meeting meetingStatusChange(String meetingFullname, String meetingStatus) {
        Meeting meetingToChange = meetingRepository.findByMeetingfullname(meetingFullname);
        String username = meetingToChange.getChairname();
        User user = userRepository.findByUsername(username);
        meetingToChange.setMeetingStatus(meetingStatus);

        //如果会议通过
        if(meetingStatus.equals("adminAgree")) {
            //向用户-会议-权限中间表中加入用户在该会议的Chair身份信息
            UserMeetingAuthPair userMeetingAuthPair = new UserMeetingAuthPair(username, meetingFullname);
            MeetingAuthority chairAuthority = meetingAuthorityRepository.findByMeetingAuthority("Chair");
            userMeetingAuthPair.setMeetingAuthority(chairAuthority);
            userMeetingAuthPairRepository.save(userMeetingAuthPair);

            //向用户的Meeting集合中添加此会议信息
            Set<Meeting> meetingSet = user.getMeetings();
            meetingSet.add(meetingToChange);
            user.setMeetings(meetingSet);
            userRepository.save(user);

            //向会议的User集合中添加此用户信息
            Set<User> userSet = meetingToChange.getUsers();
            userSet.add(user);
            meetingToChange.setUsers(userSet);
        }

        meetingRepository.save(meetingToChange);
        return meetingToChange;
    }
```

```java
ConfController.java
/*用户处理PC member邀请*/

	public void UserDealWithInvitation(String meetingfullname, String sendingDate, String invitationStatus) {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        User user = userRepository.findByUsername(username);
        MeetingInvitation toDoinvitation = meetingInvitationRepository.
                findBySendingDateAndMeetingfullnameAndReceiverUsernameAndInvitationStatus(sendingDate, meetingfullname,username, "ToBeConfirmed");

        if(invitationStatus.equals("PCAgree")) {
            //向用户-会议-权限中间表中加入用户在该会议的PC身份信息
            UserMeetingAuthPair userMeetingAuthPair = new UserMeetingAuthPair(username, meetingfullname);
            Meeting meeting = meetingRepository.findByMeetingfullname(meetingfullname);
            MeetingAuthority pcAuthority = meetingAuthorityRepository.findByMeetingAuthority("PC Member");
            userMeetingAuthPair.setMeetingAuthority(pcAuthority);
            userMeetingAuthPairRepository.save(userMeetingAuthPair);

            //若用户的会议集合没有此会议，向用户的Meeting集合中添加此会议信息
            Set<Meeting> meetingSet = user.getMeetings();
            if(!meetingSet.contains(meeting)){
                meetingSet.add(meeting);
                user.setMeetings(meetingSet);
                userRepository.save(user);
            }

            //若会议的用户集合没有此用户，向会议的User集合中添加此用户信息
            Set<User> userSet = meeting.getUsers();
            if(!userSet.contains(user)){
                userSet.add(user);
                meeting.setUsers(userSet);
                meetingRepository.save(meeting);
            }
        }

        toDoinvitation.setInvitationStatus(invitationStatus);
        meetingInvitationRepository.save(toDoinvitation);
    }
```

除此之外，我还对文件结构进行了整理，将与会议有关的操作与此前Lab2操作区分开来，将Service和Controller都分为Auth/Conf/Paper三类，以保证后端文件的有序性。



##### 实验感想

在这一次的实验之中，我最大的感受就是：持续化的开发和沟通真的十分重要。吸取了上一次lab的教训和此前优秀小组展示的经验，我们在开发过程中开展了三次以上大型会议，我与赵一玲每天都保持在微信上进行文字和语音沟通，此外还在小组群中与前端同学在每完成一个新功能后进行交互沟通；我们每天都会进行开发进度记录，并且有阶段性的目标和开发安排，没有人处于无事可做的状态之中。我们的开发属于敏捷开发，同时这种敏捷开发并不是无序的，我们每天都有相应的任务分配和积极的交流，这使得我们每天都有小任务，但小任务并不是难以忍受的。例如，这是我在后期对于前一次测试出现问题的总结，以及下一阶段开发的内容摘要：

```
meetingInvitation类加一个receiverFullname
搜索的改进功能 开启投稿
日期bug的修复 直接使用string类型不用Date类可以吗？
前端对于每个失败的请求要读取后端返回的httpstatus，然后弹窗
前端要避免用户可以通过url直接访问部分网址（利用路由的重定向？我不是很清楚这个机制，axios可以对单个页面重定向吗？）
开启投稿 /chair/startContribute
前端发送meetingFullname即可，后端meeting类加一个boolean IsContributeStart，建立会议时就设置默认为false，接受以后改为true，返回一个status:success即可
投稿页面 /user/contribute
前端发送meetingFullname Title Summary File（要求你们自己看助教文档），后端建立文件仓库和相应的request类，仓库里面存放meetingFullname Title Summary File username 以及meeting user
实现投稿以后用户直接成为author
```

在这次实验之中，我发现自己对未知问题的解决能力又上升了一个台阶。从用户对于不同会议存在不同权限问题的解决，到对于这种权限的过滤控制机制，我思考了很多方法，也发现自己在这一阶段可以算是把它解决得相对完美了。同时，我发现通过整理文件结构，自己对于代码和文件结构的规范性的认识也更近了一步：软件开发是一门艺术，我们不仅要把功能实现，而且要能够让文件层次鲜明，对于不同功能模块进行区分，以便于小组同伴进行维护与检查，否则在debug阶段检查错误出处时会出现一定困难。此外，通过最初独立进行功能分析并编写文档（见上文《第一次会议对功能进行分析》），以及与小组成员交流，我发现自己在处理任务时的条理性也变强了。这证明软件工程的确是一门实践性很强的学科，通过课程实验的捶打，我的开发能力一定会迅速提升。
