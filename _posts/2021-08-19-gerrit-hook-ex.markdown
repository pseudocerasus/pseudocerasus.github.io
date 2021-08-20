---
layout: post
title:  "Gerrit hook 샘플"
date:   2014-04-16 00:06:10 +0900
---
이제는 사용할 일이 많지 않은 Gerrit hook 샘플입니다.

```bash
#!/bin/bash
 
if [ -f debug.log ];
then
    rm debug.log
fi
touch debug.log
 
LOG=debug.log
 
OPTS=`getopt -o c:i:u:p:b:t:l:m:s: -l change:,is-draft:,change-url:,project:,branch:,topic:,uploader:,commit:,patchset: -- "$@"`
if [ $? != 0 ]
then
    echo "getopt error" >> $LOG
    cat $LOG | mail -s "Gerrit hook parameter is not valid" your_account@epicycle.uni
    exit 1
fi
 
eval set -- "$OPTS"
 
COMMIT=00000000FFFFFFFF
 
while true ; do
    case "$1" in
        --change)
          CHANGE=$2
          shift 2;;
 
        --is-draft)
          IS_DRAFT=$2
          shift 2;;
 
        --change-url)
          CHANGE_URL=$2
          shift 2;;
 
        --project)
          PROJECT=$2
          shift 2;;
 
        --branch)
          BRANCH=$2
          shift 2;;
 
        --topic)
          TOPIC=$2
          shift 2;;
        --uploader)
          UPLOADER=$2
          shift 2;;
        --commit)
          COMMIT=$2
          shift 2;;
        --patchset)
          PATCHSET=$2
          shift 2;;
        --) shift
            break;;
    esac
done
echo "CHANGE $CHANGE IS_DRAFT $IS_DRAFT CHANGE_URL $CHANGE_URL PROJECT $PROJECT BRANCH $BRANCH TOPIC $TOPIC UPLOADER $UPLOADER COMMIT $COMMIT PATCHSET $PATCHSET" >> $LOG
git log $COMMIT -1 >> $LOG
IS_REVERT_COMMIT=`git log $COMMIT -1 | grep "This reverts commit" | wc -l`
TYPE="UNKNOWN"
STATE="INIT"
for CH in `git log $COMMIT -1 | grep -A 4 Author: | grep - | sed s/Revert//g | grep -o -e .` ; do
    ABC=0
    NUM=0
    DASH=0
    TYPE="UNKNOWN"
    ABC=`echo $CH | sed -n /[a-zA-Z]/p | wc -l`
    NUM=`echo $CH | sed -n /[0-9]/p | wc -l`
    DASH=`echo $CH | grep - | wc -l`
    if [ $ABC -eq 1 ];
    then
        TYPE="CHAR"
    elif [ $NUM -eq 1 ];
    then
        TYPE="NUM"
    elif [ $DASH -eq 1 ];
    then
        TYPE="DASH"
    else
        TYPE="UNKNOWN"
    fi
    case "$TYPE" in
        CHAR)
            if [ $STATE == "INIT" -o $STATE == "START" ];
            then
                STRING+=$CH
                STATE="START"
            elif [ $STATE == "INPROGRESS" ];
            then
                STATE="FOUND"
            fi
        ;;
        NUM)
            if [ $STATE == "START" ];
            then
                STATE="ERROR"
            elif [ $STATE == "INPROGRESS" ];
            then
                STRING+=$CH
                STATE="INPROGRESS"
            fi
        ;;
        DASH)
            if [ $STATE == "START" ];
            then
                STRING+=$CH
                STATE="INPROGRESS"
            elif [ $STATE == "INPROGRESS" ];
            then
               STATE="ERROR"
            fi
        ;;
        *)
           if [ $STATE == "START" ];
           then
               STATE="ERROR"
           elif [ $STATE == "INPROGRESS" ];
           then
               STATE="FOUND"
           fi
        ;;
    esac
 
    echo "$CH" "$STRING" "$STATE" >> $LOG
done
 
JIRA_KEY=""
 
if [ $STATE == "FOUND" ];
then
    JIRA_KEY=$STRING
else
    echo "Can not find unique JIRA KEY" >> $LOG
    UPLOADER_EMAIL=`ssh -p 29418 source.epicycle.uni -l your_account.kim gerrit query $CHANGE | grep email: | sed s/^.*email:\ //g`
    cat $LOG | mail -s "Gerrit Warning - Missing JIRA KEY in your commit" $UPLOADER_EMAIL -c your_account.kim@epicycle.uni
    #ssh -p 29418 your_account@source.epicycle.uni gerrit review $COMMIT --code-review -2 --message "Missing\ a\ JIRA\ KEY\ on\ the\ commit\ message"
    exit 0
fi
 
if [ -z $JIRA_KEY ];
then
    echo "ERROR !!! JIRA KEY IS NOT VALID" >> $LOG
    cat $LOG | mail -s "Gerrit Error - Caution!!! LOOK THIS ERROR" your_account.kim@epicycle.uni
    #ssh -p 29418 your_account@epicycle.uni gerrit review $COMMIT --code-review -2 --message "Missing\ a\ JIRA\ KEY\ on\ the\ commit\ message"
    exit 0
fi
 
curl --user your_account.kim:PASSWORD http://issue.epicycle.uni/issue/rest/api/latest/search?jql=issueKey=$JIRA_KEY > jira.key
 
JIRA_KEY_VERIFY=`jshon -k < jira.key | grep issues | wc -l`
 
if [ $JIRA_KEY_VERIFY -eq 1 ];
then
    echo "JIRA KEY: $JIRA_KEY" >> $LOG
else
    echo "NON VALID JIRA KEY" >> $LOG
    cat $LOG | mail -s "Gerrit Error - Caution!!! LOOK THIS ERROR" your_account.kim@epicycle.uni
    rm jira.key
    #ssh -p 29418 your_account@source.epicycle.uni gerrit review $COMMIT --code-review -2 --message "Missing\ a\ JIRA\ KEY\ on\ the\ commit\ message"
    exit 0
fi
 
JIRA_STATUS=`jshon -e issues -a -e fields -e status -e name < jira.key`
JIRA_STATUS=`echo "$JIRA_STATUS" | sed s/\"//g`
echo "JIRA STATUS: $JIRA_STATUS" >> $LOG
 
if [ -f jira.key ];
then
    rm jira.key
fi
 
if [ ! -z $JIRA_KEY ];
then
    echo "JIRA KEY IS VAILD" >> $LOG
    ssh -p 29418 your_account.kim@source.epicycle.uni gerrit review $COMMIT --code-review 1 --message "http://issue.epicycle.uni/issue/browse/$JIRA_KEY"
 
    GERRIT_ID=`echo $CHANGE_URL | cut -d '/' -f 4`
    echo "GERRIT ID: $GERRIT_ID" >> $LOG
 
    touch $GERRIT_ID.json
    echo "{" >> $GERRIT_ID.json
    echo "\"body\": \"$CHANGE_URL\n$PROJECT\n$BRANCH\"" >> $GERRIT_ID.json
    echo "}" >> $GERRIT_ID.json
 
    curl -D- -u your_account.kim:PASSWORD -X POST --data "@$GERRIT_ID.json" -H "Content-Type: application/json" http://issue.epicycle.uni/issue/rest/api/2/issue/$JIRA_KEY/comment >> $LOG
 
    if [ -f $GERRIT_ID.json ];
    then
        rm $GERRIT_ID.json
    fi
 
else
    echo "NON VALID JIRA KEY" >> $LOG
    #ssh -p 29418 your_account@source.epicycle.uni gerrit review $COMMIT --code-review -2 --message "Missing\ a\ JIRA\ KEY\ on\ the\ commit\ message"
    exit 0
fi
 
cat $LOG | mail -s "Gerrit JIRA KEY PASS - $COMMIT" your_account.kim@epicycle.uni
```