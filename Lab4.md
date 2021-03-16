#### 后端代码检查截图

![codeCheck](img\codeCheck.png)



#### 后端tag截图


![tag3](img\lab3tag.png)

![tag4](img\lab4tag.png)



#### 后端代码最终测试结果截图


![test](img\test.png)



#### 开发过程中存在的问题总结



##### 问题描述

前端直接传入paper的作者对象，后端无法读取

##### 解决方法

前端传入json字符串，后端使用jackson转为对象。

```java
        //将json字符串转化为对象，再存入List
        ObjectMapper objectMapper = new ObjectMapper();
        AuthorOfPaper[] authorOfPapers = objectMapper.readValue(authorOfPaperList, AuthorOfPaper[].class);
        List<AuthorOfPaper> resultList = new ArrayList<>(authorOfPapers.length);
        for (AuthorOfPaper item : authorOfPapers) {
            resultList.add(item);
        }
```

##### 问题描述

后端测试可以通过，但交互时审稿一直失败

##### 解决方法

通过检查，发现后端并未将userMeetingTopicPairRepository在PaperService.java中赋初值，因此交互时缺失对象，添加即可。

##### 问题描述

后端返回前端Comment对象过长，且存在循环

##### 解决方法

通过检查，发现后端在Comment对象中有Paper类集合，Paper类中有Meeting类集合，会造成无限循环，因此需要把Meeting类中的Paper设为JsonIgnore。

##### 问题描述

前端传入后端文件大小超过1M会报错；Abstract过长也会报错

##### 解决方法

后端修改application.properties的配置，规定文件大小限制；后端在数据库内元素@Column设置length限制。

```java
#文件大小限制
spring.servlet.multipart.maxFileSize = 5MB
spring.servlet.multipart.maxRequestSize=5MB
spring.servlet.multipart.max-file-size = 5MB
spring.servlet.multipart.max-request-size = 5MB
```



#### 段欣然 18307130295


##### 实验任务

这次实验中，我主要负责后端的交互部分以及部分功能的实现，其中包括对后端源代码打标签，修改后端文件结构，使admin功能与user功能分离；编写交互及功能文档，约定所有的交互接口；编写新增的topic类、仓库以及与会议和投稿相关的topic功能代码，根据需求增加User-Meeting-Topic中间表；编写功能代码实现AuthorOfPaper类并与前端实现自定类的交互传输；编写会议开启审稿及发布功能代码；编写分配稿件方法1（基于topic的分配）功能代码；编写返回PC member分配到的待审稿件功能代码，并编写所有与自己的任务相关单元测试代码，使用postman进行测试与debug，规定接口相关信息编写所有接口功能代码。除此之外，我还在第一次会议前以任务为单位分析功能并整理成文档，部署后端项目，以及部署完毕后进行交互与功能测试及debug。



##### 实验部分核心功能代码一览

首先，第一步是吸取上一次的经验教训，把后端文件结构重构，避免文件太多造成混乱。

<img src="img\docStruct.png" alt="docStruct" style="zoom:67%;" />

例如，在这里我们将Request细分，按照角色分为AdminRequest和UserRequest两个package，在UserRequest下，又分为一般Request，以及和会议相关的Request、和Paper相关的Request，便于项目管理。

之后，我编写了Lab4的前端与后端接口约定，规定了各个状态的表示：

```java
lab4前后端接口约定 By 段欣然

注册
@PostMapping("/register") //接口名以及方式：POST
RegisterRequest{ username, password, fullname, mailbox, region, area }//request类接收的对象
response.put("id",id); //返回前端的数据

登录
@PostMapping("/login")
LoginRequest{ username, password }
response.put("token", message);
response.put("userDetails", targetUser);

url健康测试
@GetMapping("/welcome")

会议注册
@PostMapping("/user/meetingRegister")
MeetingRegisterRequest{ meetingName, meetingFullname, holdtimestart, holdtimeend, holdplace, deadline, publishtime, topics } //更新部分：这里加一个topic数组，后端接收的形式为List<String>
return ResponseEntity.ok(confService.meetingRegister(request))

管理员获取所有待审批会议
@PostMapping("/admin/stuff")
//无requestBody
response.put("size",toBeApplied.size());
response.put("applicationsTodo",toBeApplied);
//修改：返回的List中的meeting对象里面有topic数组，即这个topic

管理员审批会议
@PostMapping("/admin/workspace")
MeetingStatusChangeRequest{ meetingFullname, meetingStatus }
//更新部分：这里如果申请通过，申请者自动负责所有topic
response.put("changedMeeting",changedMeeting);

用户查看自己的会议申请
@PostMapping("/user/myMeetingApplication")
response.put("size",myapplies.size());
response.put("applications",myapplies);

用户查看自己的会议
@PostMapping("/user/myMeeting")
response.put("size",myMeetings.size());
response.put("myMeetings",myMeetings);

用户查看自己收到的会议邀请
@PostMapping("/user/myInvitation")
response.put("size",myReceivedInvitationList.size());
response.put("myReceivedInvitations",myReceivedInvitationList);
response.put("topics",topics );//更新部分：这里会增加一个topic数组作为返回值
//修改：myReceivedInvitations中每条邀请下面有一个meeting字段对象，在这个meeting对象里面有topic数组

用户处理自己收到的会议邀请
@PostMapping("/user/dealWithInvitation")
UserDealWithInvitationRequest{ meetingFullname, sendingDate, invitationStatus, chosenTopics }//更新部分：这里加一个chosenTopics数组，后端接收的形式为List<String>，chosenTopics前端需要控制非空
response.put("status","Success");

用户查看所有通过审批会议
@PostMapping("/user/getAllMeeting")
response.put("appliedMeetingList", meetingList);

Chair搜索用户
@PostMapping("/user/chair/searchUser")
MeetingChairSearchUserRequest{ meetingFullname, fullnamePart }
response.put("userList", userList);

Chair邀请PC member
@PostMapping("/user/chair/invitePC")
MeetingInvitationRequest{ meetingfullname, sendingDate, List<String> receivers }
response.put("status","Success");

Chair查看已发送邀请
@PostMapping("/user/chair/invitation")
ChairGetSentInvitationsRequest{ meetingFullname }
response.put("size", invitationList.size());
response.put("invitationList",invitationList);

Chair查看所有PC member
@PostMapping("/user/chair/getPC")
ChairGetMeetingAllPCmemberRequest{ meetingFullname }
response.put("size", PCList.size());
response.put("PCList", PCList);

Chair删除PC member
@PostMapping("/user/chair/deletePC")
ChairDeletePCmemberRequest{ pcname, meetingFullname }
response.put("status","Success");//更新部分：删除PC后也删除其与负责topic的联系

Chair开启投稿
@PostMapping("/user/chair/startContribute")
ChairStartContributeRequest{ meetingFullname }
response.put("status","Success");

用户投稿
@PostMapping("/user/contribute")
contributeFile{ title, authorOfPaperList, topicList, summary, file, meetingFullname }
response.put("status","Success");
//更新部分：前端界面增加 topic 复选框，增加作者信息输入框，前端增加一个 topic 变量（数组），一个 authorOfPaperList 对象数组   {    fullname:    mailbox:    region:    unit:   }，authorOfPaperList, topicList前端需要控制非空

用户获取当前会议所有topics
@PostMapping("/user/getTopicsOfMeeting")
getTopicsOfMeeting{ meetingFullname }
response.put("topicList",topicList);

Author查看自己的投稿
@PostMapping("/user/author/getContribution")
AuthorGetContributionRequest{ meetingFullname }
response.put("paperList", paperList);

Author获取文件
@PostMapping("/user/author/getFile")
AuthorGetFileRequest{ id, meetingFullname }
return export(file);

Author进入更新页面返回信息
@PostMapping("/user/author/enterUpdatePage")
AuthorEnterUpdatePageRequest{ id, meetingFullname }
response{ paper {id, title, authorOfPaperList, topicList, summary, fileName, meetingFullname} chosenTopics }

Author更新自己的投稿
@PostMapping("/user/author/updateContribution")
updateContribute{ id, title, authorOfPaperList, topicList, summary, file, meetingFullname }
response.put("status","Success");

Chair开启审稿
@PostMapping("/user/chair/startAudition")
ChairStartAuditionRequest{ meetingFullname, method }
response.put("status","Success");
//如果会议处于审稿状态，用户不能对稿件进行修改
//如果失败的话会返回提示信息

PC member查看分配到的稿件
@PostMapping("/user/pcmember/allocatedPaper")
PCmemberGetAllocatedPaperRequest{ meetingFullname }
response.put("size", paperList.size());
response.put("paperList", paperList);

PC member审核稿件
@PostMapping("/user/pcmember/auditPaper")
PCmemberAuditPaperRequest{ id,  score, remark, confidence }
response.put("status","Success");

Chair查看评审结果
@PostMapping("/user/chair/showResult")
response.put("size", paperList.size());
response.put("paperList", paperList);

Chair发布评审结果
@PostMapping("/user/chair/publish")
ChairPublishRequest{ meetingFullname }
response.put("status","Success");


会议申请状态表示 meetingStatus
会议提交申请：ToBeApplied
管理员通过/拒绝：adminAgree/adminDisagree

会议状态表示 contributeStatus
会议通过：Approved
会议开启投稿：Contributing
会议审稿阶段：Auditing
会议发布：Releasing

邀请各状态表示 InvitationStatus
Chair提交邀请：ToBeConfirmed
PC同意/拒绝邀请：PCAgree/PCRejected

稿件状态表示 paperStatus
稿件投递：ToBeVerified
稿件审核中：Auditing
稿件审核完毕：Audited （三个PC都审核完毕）

当前PC稿件审核状态表示 auditStatus
稿件未审核：unaudited
稿件审核：audited
```

之后，我实现了与新增的topic类有关的部分功能，以会议注册时加入topic为例，这里将topic传入单独的方法，然后在数据库中寻找，如果数据库不存在该topic则新增。

```java
ConfService.java
/*添加或取出topic*/
    
	//Xinran Duan's work
    public Set<Topic> addTopicsOfMeeting(List<String> topics, String meetingFullname){
        Iterator<String> iterator = topics.iterator();
        Set<Topic> rtntopics = new HashSet<>();
        String topicName;

        while(iterator.hasNext()) {
            topicName = iterator.next();
            if(null == topicRepository.findByTopic(topicName)) {
                Topic topic = new Topic(topicName);
                topicRepository.save(topic);
            }
            rtntopics.add(topicRepository.findByTopic(topicName));
        }

        return rtntopics;
    }
```

为了解决用户和topic之间的负责关系，我们增加了用户-会议-topic中间表。我们重构了通过会议申请的方法：会议申请通过后，申请人自动变为Chair与PC，并负责所有topic。

```java
ConfService.java
/*admin处理会议申请*/

    //Xinran Duan's work
    public Meeting meetingStatusChange(MeetingStatusChangeRequest request) {
        String meetingFullname = request.getMeetingFullname();
        String meetingStatus = request.getMeetingStatus();

        Meeting meetingToChange = meetingRepository.findByMeetingfullname(meetingFullname);
        String username = meetingToChange.getChairname();
        User user = userRepository.findByUsername(username);
        meetingToChange.setMeetingStatus(meetingStatus);

        //如果会议通过
        if(meetingStatus.equals("adminAgree")) {
            //向用户-会议-权限中间表中加入用户在该会议的Chair身份信息
            UserMeetingAuthPair userMeetingAuthPairChair = new UserMeetingAuthPair(username, meetingFullname);
            MeetingAuthority chairAuthority = meetingAuthorityRepository.findByMeetingAuthority("Chair");
            userMeetingAuthPairChair.setMeetingAuthority(chairAuthority);
            userMeetingAuthPairRepository.save(userMeetingAuthPairChair);

            //向用户-会议-权限中间表中加入用户在该会议的PC身份信息
            UserMeetingAuthPair userMeetingAuthPairPC = new UserMeetingAuthPair(username, meetingFullname);
            MeetingAuthority PCAuthority = meetingAuthorityRepository.findByMeetingAuthority("PC Member");
            userMeetingAuthPairPC.setMeetingAuthority(PCAuthority);
            userMeetingAuthPairRepository.save(userMeetingAuthPairPC);

            //向用户的Meeting集合中添加此会议信息
            Set<Meeting> meetingSet = user.getMeetings();
            meetingSet.add(meetingToChange);
            user.setMeetings(meetingSet);

            //向用户-会议-topic中间表中加入用户负责该topic的信息
            Set<Topic> topics = meetingToChange.getTopics();
            Iterator<Topic> iterator = topics.iterator();
            Topic tmp;
            while(iterator.hasNext()) {
                tmp = iterator.next();
                UserMeetingTopicPair userMeetingTopicPair = new UserMeetingTopicPair(username, meetingFullname, tmp);
                userMeetingTopicPairRepository.save(userMeetingTopicPair);
            }
            userRepository.save(user);

            //向会议的User集合中添加此用户信息
            Set<User> userSet = meetingToChange.getUsers();
            userSet.add(user);
            meetingToChange.setUsers(userSet);
            meetingToChange.setApproved(true);
        }

        meetingRepository.save(meetingToChange);
        return meetingToChange;
    }
```

在Controller的编写中，本次Lab实现较为特殊的是：前端需要传入AuthorOfPaper类到后端，对于类的传输，我们的解决方法是：前端为后端传入json字符串，后端使用jackson转为对象，再进行处理。

```java
PaperController.java
/*author修改投稿*/

    @PostMapping("/user/author/updateContribution")
    public ResponseEntity<?> updatePaper(@RequestParam("id")Long id, @RequestParam("title")String title,
                                             @RequestParam("authorOfPaperList")String authorOfPaperList,
                                             @RequestParam("topicList")List<String> topicList, @RequestParam("summary")String summary,
                                             @RequestParam(value="file", required=false) MultipartFile file, @RequestParam("meetingFullname")String meetingFullname) throws JsonProcessingException {
        logger.debug("author update paper");
        Map<String, Object> response = new HashMap<>();

        //将json字符串转化为对象，再存入List
        ObjectMapper objectMapper = new ObjectMapper();
        AuthorOfPaper[] authorOfPapers = objectMapper.readValue(authorOfPaperList, AuthorOfPaper[].class);
        List<AuthorOfPaper> resultList = new ArrayList<>(authorOfPapers.length);
        for (AuthorOfPaper item : authorOfPapers) {
            resultList.add(item);
        }

        paperService.revisePaper(file, title, resultList, topicList, summary, meetingFullname, id);
        response.put("status","Success");
        return ResponseEntity.ok(response);
    }
```

本次所编写的所有后端交互Controller及相应Request类，在文档中已提及，在此不表。

值得一提的是方法的复用：比如前端需要得到一个会议的所有Topic，我们编写了GetTopicsOfMeeting方法，并实现了相应接口，这个方法可以在前端多个页面进行应用。

```java
PaperService.java
/*返回该Meeting的所有topic*/

	public List<String> getTopicsOfMeeting(String meetingFullname) {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        Meeting meeting = meetingRepository.findByMeetingfullname(meetingFullname);

        List<String> topicList = new ArrayList<>();
        Set<Topic> topics = meetingRepository.findByMeetingfullname(meetingFullname).getTopics();

        Iterator<Topic> iterator = topics.iterator();
        Topic tmp;
        while(iterator.hasNext()) {
            tmp = iterator.next();
            topicList.add(tmp.getTopic());
        }
        return topicList;
    }

ConfController.java
    
    @PostMapping("/user/getTopicsOfMeeting")
    public ResponseEntity<?> userGetTopicsOfMeeting(@RequestBody GetTopicsOfMeetingRequest request) {
        logger.debug("user wants to get topics of a meeting");
        Map<String, Object> response = new HashMap<>();
        List<String> topicList = confService.getTopicsOfMeeting(request.getMeetingFullname());
        response.put("size", topicList.size());
        response.put("topicList",topicList);
        return ResponseEntity.ok(response);
    }
```

我负责实现基于Topic的稿件分配方式。我的思路是：for循环稿件，对于每一个稿件取出它的topic，把负责topic的PC找出来,加入PCOfTopicList（如果这个PC是author或者authorOfPaper，也加入
这个列表，但也单独加入一个PCOfTopicLimitedList，否则加入PCOfTopicNotLimitedList），根据PCList的size决定它是在所有人之间分配还是PCList中进行分配。所有人之间分配：随机找到不是author或者authorOfPaper，且负责稿件最少的即可；PCOfTopicList中分配：随机找到不是author或者authorOfPaper，且负责稿件最少的即可，如果PCOfTopicLimitedList.size ==PCOfTopicList.size，则转入所有人之间分配。代码如下：

```java
PaperService.java
/*基于topic分配稿件*/

    public void handOutPapersByTopic(String meetingfullname){
        Meeting meeting = meetingRepository.findByMeetingfullname(meetingfullname);
        if(!meeting.isApproved()) {
            throw new MeetingIsNotApprovedException(meetingfullname);
        }
        List<Paper> paperList = paperRepository.findByMeetingfullname(meetingfullname);
        List<UserMeetingAuthPair> pcPairs = userMeetingAuthPairRepository.findByMeetingfullnameAndMeetingAuthority(
                meetingfullname, meetingAuthorityRepository.findByMeetingAuthority("PC Member"));
        List<User> pcList = new ArrayList<>();

        for(int i = 0; i < pcPairs.size(); i++){
            String tmpname = pcPairs.get(i).getUsername();
            pcList.add(userRepository.findByUsername(tmpname));
        }

        //打乱pcList随机
        int size = pcList.size();
        Random random = new Random();
        for(int i = 0; i < size; i++) {
            // 获取随机位置
            int randomPos = random.nextInt(size);
            // 当前元素与随机元素交换
            User temp = pcList.get(i);
            pcList.set(i, pcList.get(randomPos));
            pcList.set(randomPos, temp);
        }

        /*
        *for循环：循环稿件
        *对于每一个稿件取出它的topic，把负责topic的PC找出来,加入PCOfTopicList（如果这个PC是author或者authorOfPaper，也加入这个列表，
        *但也单独加入一个PCOfTopicLimitedList，否则加入PCOfTopicNotLimitedList）
        *根据PCList的size决定它是在所有人之间分配还是PCList中进行分配
        *所有人之间分配：随机找到不是author或者authorOfPaper，且负责稿件最少的即可
        *PCOfTopicList中分配：随机找到不是author或者authorOfPaper，且负责稿件最少的即可，如果PCOfTopicLimitedList.size ==PCOfTopicList.size，则转入所有人之间分配 
        */

        for(int i = 0; i < paperList.size(); i++){
            Paper nowPaper = paperList.get(i);
            Set<Topic> topics = nowPaper.getChosenTopics();

            List<AuthorOfPaper> authorOfPaperList = nowPaper.getAuthorOfPapers();

            Iterator<Topic> iterator = topics.iterator();
            List<User> PCOfTopicList = new ArrayList<>();
            List<User> PCOfTopicLimitedList = new ArrayList<>();
            List<User> PCOfTopicNotLimitedList = new ArrayList<>();

            //寻找该Paper每一个topic的负责pc，维护上述3个List
            Topic tmpTopic;
            while(iterator.hasNext()) {
                tmpTopic = iterator.next();
                System.out.println(tmpTopic.getTopic());
                System.out.println(meetingfullname);
                List<UserMeetingTopicPair> topicPairList = userMeetingTopicPairRepository.findByTopicAndMeetingfullname(
                        tmpTopic, meetingfullname);
                Iterator<UserMeetingTopicPair> iter = topicPairList.iterator();
                UserMeetingTopicPair tmpPair;

                while(iter.hasNext()) {
                    tmpPair = iter.next();
                    String username = tmpPair.getUsername();
                    User pcinPair = userRepository.findByUsername(username);
                    if(!PCOfTopicList.contains(pcinPair)) PCOfTopicList.add(pcinPair);

                    AuthorOfPaper pcAuthor = authorOfPaperRepository.findByFullnameAndMailbox(pcinPair.getFullname(), pcinPair.getMailbox());
                    if((pcAuthor != null && authorOfPaperList.contains(pcAuthor)) || nowPaper.getAuthor().getUsername().equals(username) ) {
                        if(!PCOfTopicLimitedList.contains(pcinPair)) PCOfTopicLimitedList.add(pcinPair);
                    } else if(!PCOfTopicNotLimitedList.contains(pcinPair)) PCOfTopicNotLimitedList.add(pcinPair);
                }
            }

            //在当前PCOfTopicList中没有限制的PC中分配
            if(PCOfTopicNotLimitedList.size() >= 3) {
                for( int j = 0 ; j < 3 ; j++ ) {
                    Iterator<User> iter = PCOfTopicNotLimitedList.iterator();
                    User tmpUser;
                    String minUsername = "";
                    int minNum = 2147483647;

                    while(iter.hasNext()) {
                        tmpUser = iter.next();
                        PCMeetingPaperPair tmpPMPair = pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(
                                tmpUser.getUsername(), meetingfullname);
                        //如果有一个PC没有分配到稿件，直接分配给该PC
                        if(tmpPMPair == null) {
                            minNum = 0;
                            minUsername = tmpUser.getUsername();
                            break;
                        } else if(tmpPMPair.getPapers().size() < minNum && !tmpPMPair.getPapers().contains(nowPaper)) {
                            minNum = tmpPMPair.getPapers().size();
                            minUsername = tmpUser.getUsername();
                        }
                    }

                    if(minUsername.equals("")) {
                     pcMeetingPaperPairRepository.deleteAll(pcMeetingPaperPairRepository.findByMeetingfullname(meetingfullname));
                        throw new CannotHandOutEquallyException(meetingfullname);
                    }

                    PCMeetingPaperPair PMPair = pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(
                            minUsername, meetingfullname);
                    //如果有一个PC没有分配到稿件，直接分配给该PC

                    if(PMPair == null) {
                        Set<Paper> paperSetOfPC = new HashSet<>();
                        paperSetOfPC.add(nowPaper);
                        PCMeetingPaperPair newPMPair = new PCMeetingPaperPair(minUsername, meetingfullname, paperSetOfPC);
                        pcMeetingPaperPairRepository.save(newPMPair);
                    } else {
                        //否则选取稿件最少的PC为他分配
                        Set<Paper> paperSetOfPC = PMPair.getPapers();
                        paperSetOfPC.add(nowPaper);
                        pcMeetingPaperPairRepository.save(PMPair);
                    }
                }

            } else { //在所有和这个paper无关的PC中分配
                List<User> pcOKList = new ArrayList<>();
                Iterator<User> iterr = pcList.iterator();
                User tmpUser;

                while(iterr.hasNext()) {
                    tmpUser = iterr.next();
                    if(PCOfTopicLimitedList.contains(tmpUser))
                        continue;
                    else pcOKList.add(tmpUser);
                }

                for( int j = 0 ; j < 3 ; j++ ) {
                    Iterator<User> iter = pcOKList.iterator();
                    String minUsername = "";
                    int minNum = 2147483647;

                    while(iter.hasNext()) {
                        tmpUser = iter.next();
                        PCMeetingPaperPair tmpPMPair = pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(
                                tmpUser.getUsername(), meetingfullname);
                        //如果有一个PC没有分配到稿件，直接分配给该PC
                        if(tmpPMPair == null) {
                            minNum = 0;
                            minUsername = tmpUser.getUsername();
                            break;
                        } else if(tmpPMPair.getPapers().size() < minNum && !tmpPMPair.getPapers().contains(nowPaper)) {
                            minNum = tmpPMPair.getPapers().size();
                            minUsername = tmpUser.getUsername();
                        }
                    }

                    if(minUsername.equals("")) {
                        pcMeetingPaperPairRepository.deleteAll(pcMeetingPaperPairRepository.findByMeetingfullname(meetingfullname));
                        throw new CannotHandOutEquallyException(meetingfullname);
                    }

                    PCMeetingPaperPair PMPair = pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(
                            minUsername, meetingfullname);
                    //如果有一个PC没有分配到稿件，直接分配给该PC
                    if(PMPair == null) {
                        Set<Paper> paperSetOfPC = new HashSet<>();
                        paperSetOfPC.add(nowPaper);
                        PCMeetingPaperPair newPMPair = new PCMeetingPaperPair(minUsername, meetingfullname, paperSetOfPC);
                        pcMeetingPaperPairRepository.save(newPMPair);
                    } else {
                        //否则选取稿件最少的PC为他分配
                        Set<Paper> paperSetOfPC = PMPair.getPapers();
                        paperSetOfPC.add(nowPaper);
                        pcMeetingPaperPairRepository.save(PMPair);
                    }
                }
            }
        }

        //all paper 分配完毕
    }
```

此外，在设计开启审稿方法时，我也注意到多重限制。

```java
PaperService.java
/*chair开启审稿*/

	public void chairStartAudition(String meetingfullname, Long method) {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        if(null == userMeetingAuthPairRepository.findByUsernameAndMeetingfullnameAndMeetingAuthority(username,meetingfullname, meetingAuthorityRepository.findByMeetingAuthority("Chair"))){
            throw new NotHaveChairAuthorityException(username, meetingfullname);
        }

        Meeting meeting = meetingRepository.findByMeetingfullname(meetingfullname);
        if(!meeting.isApproved()) {
            throw new MeetingIsNotApprovedException(meetingfullname);
        }

        if(!meeting.isContributed()) {
            throw new MeetingIsNotContributingException(meetingfullname);
        }

        //至少要有一篇投稿才能开启审稿
        if(paperRepository.findByMeetingfullname(meetingfullname).size() == 0)
            throw new MeetingHasNoContributionException(meetingfullname);

        //除了chair还有3位pc member才能开启审稿
 if(userMeetingAuthPairRepository.findByMeetingfullnameAndMeetingAuthority(meetingfullname,meetingAuthorityRepository.findByMeetingAuthority("PC Member")).size() < 3)
            throw new LackOfPCMemberException(meetingfullname);
        
        if(method == Long.valueOf(1)) handOutPapersByTopic(meetingfullname);
            else handOutPaperEqually(meetingfullname);

        //如果可以分配，为每份稿件改变状态
        List<Paper> paperList = paperRepository.findByMeetingfullname(meetingfullname);
        Iterator<Paper> iterator = paperList.iterator();
        Paper tmp;

        while(iterator.hasNext()) {
            tmp = iterator.next();
            tmp.setPaperStatus("Auditing");
            paperRepository.save(tmp);
        }

        meeting.setContributeStatus("Auditing");
        meeting.setContributed(false);
        meeting.setAudited(true);
        meetingRepository.save(meeting);
    }
```

当为PC返回分配到的稿件时，我们利用该PC的comment如果存在，稿件一定被审核过这一特性，设置稿件的auditStatus，实现了对单个PC稿件审核状态的设置与传输。

```java
PaperService.java
/*返回PC分配到的所有稿件*/

	public List<Paper> pcGetAllocatedPaper(String meetingFullname) {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        if(null == userMeetingAuthPairRepository.findByUsernameAndMeetingfullnameAndMeetingAuthority(username,
                meetingFullname, meetingAuthorityRepository.findByMeetingAuthority("PC Member"))){
            throw new NotHavePCMemberAuthorityException(username, meetingFullname);
        }

        Meeting meeting = meetingRepository.findByMeetingfullname(meetingFullname);
        if(!meeting.isApproved()) {
            throw new MeetingIsNotApprovedException(meetingFullname);
        }

        if(!meeting.isAudited()) {
            return new ArrayList<>();
        }

        PCMeetingPaperPair pcMeetingPaperPair = pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(username, meetingFullname);
        Set<Paper> paperSet = pcMeetingPaperPair.getPapers();
        List<Paper> paperList = new ArrayList<>(paperSet);
        Iterator<Paper> iterator = paperList.iterator();
        Paper tmp;

        while(iterator.hasNext()) {
            tmp = iterator.next();
            //设置当前PC的审稿状态
            if(null != paperCommentRepository.findByPaperidAndPcname(tmp.getId(), username))
                tmp.setAuditStatus("audited");
            else tmp.setAuditStatus("unaudited");
        }

        return paperList;
    }

```

对于单个Test类的设计，与上一次相同，我们需要保证字段不重复，在测试正确的基础上不干扰其他Test类的实现。如果测试失败，需要通过断点输出法排查究竟是哪一个步骤出现了问题，并进行完善或改正。对于Paper类，我们可以在不上传文件的情况下new一个Paper对象，但是需要注意一些属性的设置。

```java
PaperServiceTest.java
/*测试类编写的样例展示*/

    @Test
    public void chairStartPublish(){
        AuthService authService = new AuthService(userRepository, authorityRepository, authenticationManager,
                jwtUserDetailsService, jwtTokenUtil, passwordEncoder);
        ConfService confService = new ConfService(userRepository, authorityRepository, meetingRepository,
                meetingAuthorityRepository, jwtUserDetailsService, userMeetingAuthPairRepository,
                meetingInvitationRepository, topicRepository, userMeetingTopicPairRepository);
        PaperService paperService = new PaperService(userRepository, authorityRepository, meetingRepository,
                meetingAuthorityRepository, jwtUserDetailsService, userMeetingAuthPairRepository, meetingInvitationRepository,
                paperRepository, topicRepository, authorOfPaperRepository, userMeetingTopicPairRepository, pcMeetingPaperPairRepository,paperCommentRepository);

        RegisterRequest ar = new RegisterRequest("fgfgef", "wobushizs123", "zhangAAAsan",
                "zs1881@126.com", "China", "Shanghai");
        authService.register(ar);

        String usernameA = "fgfgef";
        String passwordA = "wobushizs123";
        authService.login(usernameA, passwordA);
        User userA = userRepository.findByUsername(usernameA);
        Set<Meeting> msa = new HashSet<Meeting>();

        List<String> topics= new ArrayList<>();
        topics.add("test");

        MeetingRegisterRequest m1 = new MeetingRegisterRequest("mee1", "testtt3", "2020-01-12",
                "2020-01-15", "Shanghai", "2020-4-18", "2020-4-29", topics);
        Meeting tm1 = confService.meetingRegister(m1);
        tm1.setMeetingStatus("Contributing");
        tm1.setApproved(true);
        tm1.setContributed(true);

        MeetingAuthority chairAuthority = meetingAuthorityRepository.findByMeetingAuthority("Chair");
        UserMeetingAuthPair atm1 = new UserMeetingAuthPair(userA.getUsername(), tm1.getMeetingFullname());
        atm1.setMeetingAuthority(chairAuthority);
        userMeetingAuthPairRepository.save(atm1);
        MeetingAuthority pcAuthority = meetingAuthorityRepository.findByMeetingAuthority("PC Member");
        UserMeetingAuthPair atm2 = new UserMeetingAuthPair(userA.getUsername(), tm1.getMeetingFullname());
        atm2.setMeetingAuthority(pcAuthority);
        userMeetingAuthPairRepository.save(atm2);

        meetingRepository.save(tm1);
        msa.add(tm1);

        userA.setMeetings(msa);
        userRepository.save(userA);

        RegisterRequest pc1r = new RegisterRequest("pccc111", "wobushizs123", "zhangBBBsan",
                "zs1991@126.com", "China", "Shanghai");
        authService.register(pc1r);
        User pc1=userRepository.findByUsername("pccc111");
        MeetingAuthority pc = meetingAuthorityRepository.findByMeetingAuthority("PC Member");
        UserMeetingAuthPair pcp1 = new UserMeetingAuthPair(pc1.getUsername(), tm1.getMeetingFullname());
        pcp1.setMeetingAuthority(pc);
        userMeetingTopicPairRepository.save(new UserMeetingTopicPair(pc1.getUsername(),"testtt3",topicRepository.findByTopic("test")));
        userMeetingAuthPairRepository.save(pcp1);
        pc1.setMeetings(msa);
        userRepository.save(pc1);

        RegisterRequest pc2r = new RegisterRequest("pccc222", "wobushizs123", "zhangBBBsan",
                "zs2001@126.com", "China", "Shanghai");
        authService.register(pc2r);
        User pc2=userRepository.findByUsername("pccc222");

        UserMeetingAuthPair pcp2 = new UserMeetingAuthPair(pc2.getUsername(), tm1.getMeetingFullname());
        pcp2.setMeetingAuthority(pc);
        userMeetingTopicPairRepository.save(new UserMeetingTopicPair(pc2.getUsername(),"testtt4",topicRepository.findByTopic("test")));
        userMeetingAuthPairRepository.save(pcp2);
        pc2.setMeetings(msa);
        userRepository.save(pc2);


        RegisterRequest br = new RegisterRequest("gggggh", "wobushizs123", "zhangBBBsan",
                "zs2111@126.com", "China", "Shanghai");
        authService.register(br);

        UserMeetingTopicPair UMTpair1 = new UserMeetingTopicPair("fgfgef", "testtt3", topicRepository.findByTopic("test"));
        userMeetingTopicPairRepository.save(UMTpair1);
        UserMeetingTopicPair UMTpair2 = new UserMeetingTopicPair("pccc111", "testtt3", topicRepository.findByTopic("test"));
        userMeetingTopicPairRepository.save(UMTpair2);

        String usernameB = "gggggh";
        String passwordB = "wobushizs123";
        authService.login(usernameB, passwordB);

        MeetingAuthority authorAuthority = meetingAuthorityRepository.findByMeetingAuthority("Author");

        List<AuthorOfPaper> authorOfPaperList = new ArrayList<>();
        AuthorOfPaper testAuthor1 = new AuthorOfPaper("xiaoming", "xiaoming@126.com", "Shanghai",
                "FDU");
        AuthorOfPaper testAuthor2 = new AuthorOfPaper("xiaogang", "xiaogang@163.com", "Shanghai",
                "SJTU");
        authorOfPaperRepository.save(testAuthor1);
        authorOfPaperRepository.save(testAuthor2);
        authorOfPaperList.add(testAuthor1);
        authorOfPaperList.add(testAuthor2);

        Paper paper1 = new Paper("test", "gggggh", "testtt3", "dxr is testing");
        paper1.setAuthor(userRepository.findByUsername("gggggh"));
        paper1.setMeeting(meetingRepository.findByMeetingfullname("testtt3"));
        paper1.setAuthorOfPapers(authorOfPaperList);
        paperRepository.save(paper1);

        Paper paper2 = new Paper("test2", "gggggh", "testtt3", "dxr is testing");
        paper2.setAuthor(userRepository.findByUsername("gggggh"));
        paper2.setMeeting(meetingRepository.findByMeetingfullname("testtt3"));
        paper2.setAuthorOfPapers(authorOfPaperList);
        paperRepository.save(paper2);

        UserMeetingAuthPair userBAuth = new UserMeetingAuthPair("gggggh", "testtt3", authorAuthority);
        userMeetingAuthPairRepository.save(userBAuth);

        User userB = userRepository.findByUsername(usernameB);
        userB.setMeetings(msa);
        userRepository.save(userB);
        userB = userRepository.findByUsername(usernameB);

        Meeting mmm = meetingRepository.findByMeetingfullname(tm1.getMeetingFullname());
        Set<User> users = mmm.getUsers();
        users.add(userB);
        mmm.setUsers(users);
        meetingRepository.save(mmm);

        //upload
        UserMeetingAuthPair userBp = new UserMeetingAuthPair(userB.getUsername(), tm1.getMeetingFullname());
        userBp.setMeetingAuthority(pc);
        userMeetingTopicPairRepository.save(new UserMeetingTopicPair(userB.getUsername(),"testtt3",topicRepository.findByTopic("test")));
        userMeetingAuthPairRepository.save(userBp);

        //测试gggggh投稿两份，pccc111投稿一份，pccc222投稿一份

        authService.login("pccc111", passwordB);
        Paper paper3 = new Paper("test3", "pccc111", "testtt3", "dxr is testing");
        paper3.setAuthor(userRepository.findByUsername("pccc111"));
        paper3.setMeeting(meetingRepository.findByMeetingfullname("testtt3"));
        paper3.setAuthorOfPapers(authorOfPaperList);
        paperRepository.save(paper3);

        UserMeetingAuthPair pc1Auth = new UserMeetingAuthPair("pccc111", "testtt3", authorAuthority);
        userMeetingAuthPairRepository.save(pc1Auth);

        authService.login("pccc222", passwordB);
        Paper paper4 = new Paper("test4", "pccc222", "testtt3", "dxr is testing");
        paper4.setAuthor(userRepository.findByUsername("pccc222"));
        paper4.setMeeting(meetingRepository.findByMeetingfullname("testtt3"));
        paper4.setAuthorOfPapers(authorOfPaperList);
        paperRepository.save(paper4);

        UserMeetingAuthPair pc2Auth = new UserMeetingAuthPair("pccc222", "testtt3", authorAuthority);
        userMeetingAuthPairRepository.save(pc2Auth);

        System.out.println(userRepository.findByUsername("pccc111").getPapers().size()+" +"+userRepository.findByUsername("pccc111").getPapers());
        System.out.println(userRepository.findByUsername("pccc222").getPapers().size());
        System.out.println(userRepository.findByUsername(userB.getUsername()).getPapers().size());

        assertEquals(4,paperRepository.findByMeetingfullname("testtt3").size());
        Long method = Long.valueOf(1);

        authService.login(usernameA, passwordA);
        paperService.chairStartAudition("testtt3", method);

        System.out.println(pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname("fgfgef","testtt3").getPapers().size());
        System.out.println(pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname("pccc111","testtt3").getPapers().size());
        System.out.println(pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname("pccc222","testtt3").getPapers().size());
        System.out.println(pcMeetingPaperPairRepository.findByUsernameAndMeetingfullname(usernameB,"testtt3").getPapers().size());

        authService.login("pccc111", passwordB);

        assertEquals(3, paperService.pcGetAllocatedPaper("testtt3").size());
        List papers = paperRepository.findByMeetingfullname("testtt3");
        Iterator<Paper> iterator = papers.iterator();
        Paper paper;
        Long testid;

        while (iterator.hasNext()) {
            paper = iterator.next();
            testid = paper.getId();
            List<PCMeetingPaperPair> pairs=pcMeetingPaperPairRepository.findByMeetingfullname(paper.getMeetingfullname());
            User testpc;
            for(int i=0;i<pairs.size();i++){
                if(pairs.get(i).getPapers().contains(paper)){
                    System.out.println("index"+ i +pairs.get(i).getUsername());
                    testpc = userRepository.findByUsername(pairs.get(i).getUsername());
                    authService.login(userRepository.findByUsername(pairs.get(i).getUsername()).getUsername(),"wobushizs123");
                    paperService.pcAuditPaper(paper.getId(),-1l,1l,"all pc finish test");
                    System.out.println(paperRepository.findById(paper.getId()).get().getComments().size());
                }
            }
        }

        usernameA = "fgfgef";
        passwordA = "wobushizs123";
        authService.login(usernameA, passwordA);

        List<Paper> paperList = paperService.chairGetAllPaper("testtt3");
        System.out.println(paperList.size());
        Iterator<Paper> iter = paperList.iterator();
        Paper tmp;

        while(iter.hasNext()) {
            tmp = iter.next();
            System.out.println(tmp.getPaperStatus());
        }

        paperService.chairStartPublish("testtt3");

    }
```

为了解决PC member在不同会议中分配到不同topic的问题，我创建了一个新的实体类：UserMeetingTopicPair类，这个实体类除了id元素，还有三个字段：username、meetingfullname、topic，这三个字段的联合查询可保证结果唯一，而依赖单个字段查询可以返回与单个用户/单个会议相关的所有topic信息。

```java
UserMeetingTopicPair.java
/*存储用户在不同会议中对应不同PC负责的topic的实体类，username+meetingfullname+topic三个字段联合查询，可保证结果唯一*/

package fudan.se.lab2.domain.Pair;

import com.fasterxml.jackson.annotation.JsonIgnore;
import fudan.se.lab2.domain.Topic;

import javax.persistence.*;
import java.io.Serializable;

@Entity
public class UserMeetingTopicPair{
    private static final long serialVersionUID = -8963769171279943402L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String username;
    private String meetingfullname;

    @ManyToOne(cascade = CascadeType.MERGE, fetch = FetchType.EAGER)
    @JsonIgnore
    private Topic topic = new Topic();

    public UserMeetingTopicPair(){}

    public UserMeetingTopicPair(String username, String meetingfullname, Topic topic) {
        this.username = username;
        this.meetingfullname = meetingfullname;
        this.topic = topic;
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

    public void setTopic(Topic topic) {
        this.topic = topic;
    }

    public Topic getTopic() {
        return topic;
    }
}
```

我所实现的所有功能类如下所示：

```java
    //Xinran Duan's work
	public Set<Topic> addTopicsOfMeeting(List<String> topics, String meetingFullname)
    public List<String> getTopicsOfMeeting(String meetingFullname)
    public Paper authorGetOnePaper(Long id, String meetingFullname)
    public List<String> getTopicStringsOfOnePaper(Paper paper)
    public void handOutPapersByTopic(String meetingfullname)
    public void chairStartAudition(String meetingfullname, Long method)
    public List<Paper> pcGetAllocatedPaper(String meetingFullname)
    public List<Paper> chairGetAllPaper(String meetingfullname)
    public void chairStartPublish(String meetingfullname)
```

我所实现的所有实体类及Request类如下所示：

```java
	//Xinran Duan's work
	public class UserMeetingTopicPair
    public class AuthorOfPaper
	public class GetTopicsOfMeetingRequest
    public class ChairStartAuditionRequest
    public class ChairStartPublishRequest 
    public class PCmemberGetAllocatedPaperRequest
    public class AuthorGetContributionRequest
    public class AuthorEnterUpdatePageRequest
```

我所实现的所有Controller类如下所示：

```java
	//Xinran Duan's work
    @PostMapping("/user/getTopicsOfMeeting")
    @PostMapping("/user/author/getContribution")
    @PostMapping("/user/author/enterUpdatePage")
    @PostMapping("/user/chair/startAudition")
    @PostMapping("/user/pcmember/allocatedPaper")
    @PostMapping("/user/pcmember/auditPaper")
    @PostMapping("/user/chair/showResult")
    @PostMapping("/user/chair/publish")
```

我所实现的所有Exception类如下所示：

```java
	//Xinran Duan's work
	public class NotHaveViewOrDownloadPaperAuthorityException extends RuntimeException
    public class LackOfPCMemberException extends RuntimeException
    public class MeetingCannotPublishException extends RuntimeException
    public class MeetingHasNoContributionException extends RuntimeException
	public class MeetingIsNotApprovedException extends RuntimeException
	public class MeetingIsNotAuditingException extends RuntimeException        
    public class MeetingIsNotContributingException extends RuntimeException
    public class NotAuthorOfThisPaperException extends RuntimeException    
    public class MailboxHasBeenRegisteredException extends RuntimeException    
```



##### 实验感想

在这一次的实验之中，我们每一天都在持续开发并且debug，我们每天都会进行开发进度记录，并且有阶段性的目标和开发安排，有驱动开发的文档以及日记记录。虽然在假期期间进行开发比较辛苦，但是我们在前期辛苦一些，在后期的测试阶段也不至于手忙脚乱。此外，我认为我们对于未知方法的尝试，对于bug的处理能力也是很重要的。测试阶段遇到bug时，需要利用断点输出法，找出bug所在之处，之后分析样例期望和程序断点显示不同的原因，再进行代码调整；交互阶段遇到bug时，首先需要思考问题出在前端还是后端，再相应地进行修改。要善于利用搜索引擎和相关官方文档。在开发过程中，我们遇到了前端和后端细节之处的小错误各一个，但这两个小错误却拖慢了整体的进度。因此，对于代码的细致检查和方法的学习还是十分有必要的。Bug并不可怕，但是小组的同学一定要齐心协力，每位同学各司其职，才能让开发有序进行。交互过程中遇到问题时，前端和后端的相关负责同学一定要进行充分交流，统一标准，方法确定可行之后不要贸然修改。对于小细节和极限数据的测试一定要做好，否则会出现一些本该避免的错误。

