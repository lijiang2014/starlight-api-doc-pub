# models

```
// App struct , 描述应用的主要模型
type App struct {
	Id              int64  `json:",omitempty"`
	Name            string `xorm:"unique index"`
	*AppInfo        `xorm:"extends" json:",omitempty"`
	*AppInfoVerbose `xorm:"extends" json:",omitempty"`
	*AppWidgets     `xorm:"extends" json:",omitempty"`
	*AppSetting     `xorm:"extends" json:",omitempty"`
	*AppAction      `xorm:"extends" json:",omitempty"`
}

// AppInfo for Page Show， 主要是用于控制 App 在列表中显示时的信息
type AppInfo struct {
	Title       string   `json:",omitempty"` // title of app
	Published   bool     `json:",omitempty"`
	Icon        string   `json:",omitempty"`
	Group       []string `json:",omitempty"`
	GroupAccess bool     `json:",omitempty"` // true for user in group can access, false 表示关闭了 GroupAccess
	Black       int      `json:",omitempty"` // 0 表示正常(白名单机制)， 1 表示 GroupAccess 为黑名单机制
}

// AppInfoVerbose 详细信息，冗余信息，统计信息 ...
type AppInfoVerbose struct {
	//Id          int64  // pk-u
	Description string     `json:",omitempty"`        // 描述性文字
	Creator     string     `json:"creator,omitempty"` // App 创建人
	CreateAt    *time.Time `json:",omitempty" xorm:"created"`
	PublishedAt *time.Time `json:",omitempty"`
	PublishBy   string     `json:",omitempty"` // App 审核人
	SubmitCount int64      `json:",omitempty"`
	Version     int64      `json:"version, omitempty" xorm:"version"`
	UpdatedAt   *time.Time `json:",omitempty" xorm:"updated"`
}

// AppWidgets App Render Data 控制绘制 App 表单
type AppWidgets struct {
	Widgets    Widgets `json:",omitempty"`
	JavaScript string  `json:",omitempty"` // 目前已经弃用
	CSS        string  `json:",omitempty"` // 目前已经弃用
}

// AppSetting App Spooler Creation Setting App 产生的作业
type AppSetting struct {
	SpoolerCreate        bool `json:",omitempty"`
	SpoolerExpirationDay int  `json:",omitempty"`
	Profile              bool `json:",omitempty"`
	Resubmit             bool `json:",omitempty"`
	ResubmitSameSpooler  bool `json:",omitempty"`
	ResubmitResetTTL     bool `json:",omitempty"`
}


// AppAction 应用执行相关的字段，对于非应用所有者的用户，其中部分内容不对其开发访问
type AppAction struct {
	Action  string            `json:",omitempty" xorm:"text"`
	Jobfile string            `json:",omitempty" xorm:"text"`
	Target  string            `json:",omitempty"` //
	Mode    string            `json:",omitempty"` // Action Mode . hpc-job // k8s-job // session // emm
	Url     string            `json:",omitempty"` // if not null , just serve this one
	Config  map[string]string `json:",omitempty"` // Conf K-V map
	Rusage  map[string]string `json:",omitempty"` // Rusage, Reasource usage, example: hpc:true;hpc_cluster:tianhe2d;hpc_cluster_partition:tianhe2d:lava;
}
```



# APIs

### POST /app/app 创建新App

Roles: user/staff/admin



Request example:

```
{
	"spec": {
		"Name": "Spooler_test{{$timestamp}}",
		"Title": "Spooler_test",
		"Description": "Spooler_test",
		"Target": "hpc",
		"Widgets": [
                {
                    "Id": "info1",
                    "Type": "info",
                    "Attr": {
                        "innerHTML": "<h1>Jupyter </h1><br>\n\t\t\t<h3>an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. </h3>\n\t\t\t <a href='https://jupyter.org/'>homepage</a>\n\t\t\t <br> 第一次使用需设置密码 \n\t\t\t <br> 以后可以通过选择使用已有的配置来避免重设密码"
                    }
                },
                {
                    "Id": "jupyter_genpassword",
                    "Type": "boolean",
                    "Attr": {
                        "Default": "true.",
                        "Label": "设置密码",
                        "Required": false,
                        "Selected": true
                    }
                },
                {
                    "Id": "jupyter_password",
                    "Type": "password",
                    "Attr": {
                        "Default": "",
                        "Extra": "",
                        "Label": "设定密码 : ",
                        "Required": false
                    }
                },
                {
                    "Id": "jupyter_password_verify",
                    "Type": "password",
                    "Attr": {
                        "Default": "",
                        "Extra": "",
                        "Label": "密码（再输入一次） : ",
                        "Required": false
                    }
                }
            ],
        "Action": "#!/bin/sh\nif [  -n \"$jupyter_genpassword\" ];then\n   if [ \"s$jupyter_password\"  !=  \"s$jupyter_password_verify\"  ];then\n\t\techo 1>&2 password not pass verification\n\t\texit -1\n   fi\nelse\n   JUPYTER_CONF_FILE=~/.jupyter/jupyter_notebook_config.py\n   if [ -e $JUPYTER_CONF_FILE ];then\n    cat $JUPYTER_CONF_FILE | grep '^ *c.NotebookApp.password *=.*sha1:.*' &> /dev/null\n    ret2=$?\n    if [ $ret2 -ne 0 ];then\n        echo 1>&2 \"Error: Password is not set in config file: $JUPYTER_CONF_FILE. Refusing to start the server.\"\n        exit -1\n    fi\n  else\n    echo \"Error: Can not found jupyter config file.\"\n    exit -1\n  fi\n\nfi\nmodule load anaconda3\nslurmbout=$(sbatch    $1 )\nretcode=$?\nif [ $retcode -ne 0 ] ; then\nexit $retcode\nfi\necho ${slurmbout} | awk '{print $4}'\n",
        "Jobfile": "#!/bin/sh\n#SBATCH --time=24:00:00\n#source /WORK/app/toolshs/cnmodule.sh\n#module load anaconda3\nNBPORT=\"8888\"    # notebook default port\nJUPYTER_CONF_FILE=~/.jupyter/jupyter_notebook_config.py  # do not quote\n# check password\nif [ -n \"$jupyter_genpassword\"  ];then\n   echo \"create jupyter config file with password\"\n   rm $JUPYTER_CONF_FILE\n   hashedpw=$(python -c \"from notebook.auth import passwd; print(passwd('$jupyter_password'))\")\n   jupyter notebook --generate-config\n   echo \"c.NotebookApp.password = '$hashedpw'\" >> $JUPYTER_CONF_FILE\nelse\n  if [ -e $JUPYTER_CONF_FILE ];then\n\n    cat $JUPYTER_CONF_FILE | grep '^ *c.NotebookApp.password *=.*sha1:.*' > /dev/null\n    ret2=$?\n    if [ $ret2 -eq 0 ]; then\n        echo \"Pass config check, password already set in config file: $JUPYTER_CONF_FILE\"\n    else\n        echo \"Error: Password is not set in config file: $JUPYTER_CONF_FILE. Refusing to start the server.\"\n        exit -1\n    fi\n  else\n    echo \"Error: Can not found jupyter config file.\"\n    exit -1\n  fi\nfi\nmkdir ./endpoints\nEndpointsFile=./endpoints/$SLURM_JOB_ID\necho http://$HOSTNAME:$NBPORT > $EndpointsFile\ncd ~\njupyter notebook --no-browser --port=$NBPORT --ip=0.0.0.0\n",
        "Mode":"spooler",
        "Rusage": {
            "session": "http"
        }
	}
}
```

### GET /app/app(?self=true/field_mode=...) 获取App列表
Roles: user/staff/admin

### GET /app/app/{{APP_ID}} 根据id获取App信息
Roles: user/staff/admin

### GET /app/app/{{APP_NAME}} 根据name获取App信息
Roles: user/staff/admin

用户可以通过此接口根据App的名称获取具体App的具体信息

response example:

```
{
    "apiVersion": "v0.0.3",
    "kind": "app.app",
    "spec": {
        "Id": 5,
        "Name": "lammps",
        "Title": "Lammps",
        "Published": true,
        "Icon": "file://mei/7410e42b-35ff-4c0c-9960-9e8ce43649e6",
        "CreateAt": "2018-06-14T14:56:12+08:00",
        "PublishedAt": "2018-11-12T08:20:58+08:00",
        "PublishBy": "root",
        "version": 28,
        "UpdatedAt": "2018-11-12T08:20:58+08:00",
        "Widgets": [
            {
                "Id": "info1",
                "Type": "info",
                "Attr": {
                    "innerHTML": "<h1>Lammps Molecular Dynamics Simulator</h1>\n<p>test create a new one&nbsp;</p>\n<p>version IIII</p>\n<p><br /> <a href=\"http://lammps.sandia.gov/\">homepage</a></p>"
                }
            },
            {
                "Id": "BIHU_INSTANCE_NAME",
                "Type": "text",
                "Attr": {
                    "Default": "lammps",
                    "Extra": "需要的节点数",
                    "Label": "作业名称 : ",
                    "Required": true,
                    "Rules": [
                        {
                            "pattern": "^[a-z][a-z0-9_]{3,23}[a-z0-9]$"
                        }
                    ]
                }
            },
            {
                "Id": "casedir",
                "Type": "rfb",
                "Attr": {
                    "Class": "utextclass",
                    "Default": "",
                    "Extra": "(点击选择文件夹 ， 包含了需要的输入数据文件)",
                    "Label": "Case Dir : "
                }
            },
            {
                "Id": "infile",
                "Type": "rfb",
                "Attr": {
                    "Default": "",
                    "Extra": "(点击选择)",
                    "Label": "输入文件 : ",
                    "Required": true
                }
            },
            {
                "Id": "cpus",
                "Type": "text",
                "Attr": {
                    "Class": "utextclass",
                    "Default": "40",
                    "Extra": "需要的核数",
                    "Label": "CPUs : ",
                    "Required": true,
                    "Rules": [
                        {
                            "message": "必须是数字",
                            "pattern": "[1-9][0-9]*"
                        }
                    ]
                }
            },
            {
                "Id": "nodes",
                "Type": "text",
                "Attr": {
                    "Class": "utextclass",
                    "Default": "2",
                    "Extra": "需要的节点数",
                    "Label": "Nodes : ",
                    "Required": true,
                    "Rules": []
                }
            },
            {
                "Id": "version",
                "Type": "listdyn",
                "Attr": {
                    "ArgsFixed": "app=lammps&target=hpc",
                    "Class": "ulist class",
                    "Disabled": false,
                    "EmbeddedApp": "module_avail",
                    "Label": "软件版本： ",
                    "Mode": "text",
                    "Required": true
                },
                "Inner": [
                    {
                        "Type": "option",
                        "Attr": {
                            "Label": "7Dec15-icc-14.0.2",
                            "Value": "7Dec15-icc-14.0.2"
                        }
                    },
                    {
                        "Type": "option",
                        "Attr": {
                            "Label": "31Mar17-icc-14.0.2",
                            "Selected": true,
                            "Value": "31Mar17-icc-14.0.2"
                        }
                    }
                ]
            }
        ],
        "Target": "hpc",
        "Mode": "spooler"
    }
}
```

### PATCH /app/app/{{APP_ID}} 更新app

```
Roles: user/staff/admin
Request example:
{
	"spec": {
		"id": {{APP_SPOOLER_ID}},
		"Name": "Spooler_test",
		"Title": "Spooler_test",
		"Description": "Spooler_test_test",
		"version": {{APP_SPOOLER_VERSION}},
		"Target": "hpc",
		"Widgets": [
            {
                "Id": "info1",
                "Type": "info",
                "Attr": {
                    "innerHTML": "<h1>dcv</h1>\n<b> 经过简单的配置来启动您的远程桌面 \n<br> 您需要设置密码来保障你的链接的安全性\n<br> 您可以设置旁观密码来将您的远程桌面分享给旁观者\n<br> 旁观者只能查看您的远程桌面无法进行操作  \n<br> 提交后，等待创建后DCV环境后，下载生成的 *.vnc 文件用 DCV 客户端打开即可\n<br> 若您没有 DCV 客户端，可以去 <a href=\"https://www.nice-software.com/products/dcv\">Homepage</a> 下载您需要的版本。\n<br> 使用完请主动关闭，为了避免您遗忘，应用设置了最长运行时间100小时。\n</b>"
                }
            },
            {
                "Id": "dcv_sessionname",
                "Type": "text",
                "Attr": {
                    "Default": "desktop",
                    "Extra": "",
                    "Label": "自定义会话名称 : ",
                    "Required": true
                }
            },
            {
                "Id": "dcv_passwd",
                "Type": "password",
                "Attr": {
                    "Default": "nscc2017",
                    "Extra": "",
                    "Label": "设置密码 : ",
                    "Required": true
                }
            }
        ],
        "Mode": "spooler"
	}
}
```

### DELETE /app/app/{{APP_ID}} 删除App


### POST /app/exec/{{APP_NAME}}/0 执行App

目前因工作流暂时没有落实，执行 App 都是新建 spooler id 进行执行，有工作流之后，这里应该为具体的 sid.


```
Roles: user/staff/admin
Request example:
{
	"info1": "",
	"jupyter_genpassword": "true.",
	"jupyter_password": "123123",
	"jupyter_password_verify": "123123",
	"text1": ""
}
```

这里 {{APP_NAME}} 为在 app/app 接口中注册的 App.Name 属性；Body参数为 app 的可输入变量 ， 可以为 application/json 格式 或 application/x-www-form-urlencoded 格式 或 multipart/form-data 格式 ； 需要在 Header  Content-Type  中进行说明。

