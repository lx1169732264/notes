# 需求阶段

1、简单功能产品尽量要把demo贴到issue上，复杂功能也需要把demo链接贴到issue上

2、开发在写代码的过程中对不明确的需求不要自己发散，需要与产品确认，把确认结果记录在issue上面并及时告知测试（比如新建-》活动模板）

3、测试写用例过程中若与产品沟通后有新增或变更的内容，需要把结果记录在issue上面并及时告知开发

4、需求变更、新增记录issue上需要写变更时间；需求删除使用删除线（不能直接删除）

5、开发提测时，需要保证issue上记录的内容与自己做的内容是一致并且准确的



# 提测阶段



**测试建议**

* 功能之外影响到的其他地方

* 没写到或没写清楚的地方

* 自己觉得需要着重注意的地方



**标签&脚本**

转测时需要标记label：部署模块、已解决、脚本	**（回归不通过仅由测试标记）**

需要部署的项目与模块，在测试建议中写明（https://git.darcytech.com/repos/theia/issues/283）

```
theia-pc测试分支：hotfix_283_积分加钱购功能增加风险提示
```

脚本放在tech项目的issue里，项目内issue按规范贴sql的链接，如果sql在comment里面，需要贴comment的链接

除insert之外的sql都需要review，review人写review ok（https://git.darcytech.com/repos/crm/theia-jd/issues/21）



**bug原因**

bug列表和原因序号对齐，包括未回归通过的（https://git.darcytech.com/repos/crm/theia-jd/issues/12）

写清楚bug原因，除非特别简单的文案和样式问题，禁止写“已修复”

修改的bug若**有其他影响范围，重写测试建议**，简单的可以写在bug原因里，复杂新建一个标题写



**测试环境**

理论上测试环境的Jenkins部署还有数据库修改开发不能随意操作，需要操作的话提前通知测试确认是否能操作

测试环境的数据即使要使用，也尽量使用自己新建的数据（比如互动活动）



# 测试阶段

1、测试需要监督label是否正确标记、测试建议是否写完整、sql和mergeRequest是否review。这其中任何一个不合规范，可以把issue再转回去

2、一般情况下issue和mergeRequest是同时提过来的，只提一个过来先拒绝测试

3、merge时需要记录review的人

4、测试需要打的标签

1）bug转给开发时，去除已解决标签

2）回归bug时，若有回归不通过的bug，标记回归不通过标签

3）测试完成时，标记测试完成标签

5、记录bug需要描述简洁准确，复杂bug需要写清楚测试步骤，需要把有问题的数据留存好

6、监督bug原因有没有写准确，测试需要看懂bug原因，如果看不懂，或者觉得bug原因有误，需要再与开发沟通

7、每个轮次都需要测试完整，比如第一轮测试需要把能测到的case全部测完再转给开发，减少来回测试

8、开发需要统一修改bug再转测，测试这边拒绝每个bug都部署一遍

9、测试通过后，要写测试要点和一些特殊点的标记，这个是需要开发看的，检查测试点有没有问题，有没有遗漏



# MergeRequest



标题符合规范：关联的issue号+标题，可以直接链接过去（https://git.darcytech.com/repos/crm/theia-jd/merge_requests/11）

若是其他项目的issue，写项目名+issue号（https://git.darcytech.com/repos/hermes/merge_requests/7484）

修复bug也需要提mergeRequest和review的，特别是改动比较大的bug

mergeRequest需要review，review人要写review Ok（转给测试的时候开发需要监督）





# 发布阶段



1、每次发布前测试需要执行基础用例，保障发布的版本可靠

2、是否可以发布需要由测试决定