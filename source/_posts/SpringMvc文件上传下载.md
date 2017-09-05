---
title: SrpingMVC 实现文件上传下载
date: 2017-08-23 21:42:05
tags: 学术
categories: JAVA开发
---

#### 1、文件上传功能
&nbsp;&nbsp;使用 [webuploader](http://fex.baidu.com/webuploader/) 插件，SpringMVC文件上传介绍：</br>
&nbsp;&nbsp;&nbsp;&nbsp;1、从官网下载该插件并导入项目</br>
&nbsp;&nbsp;&nbsp;&nbsp;2、引入文件到页面中<br/>
&nbsp;&nbsp;&nbsp;&nbsp;3、初始化该插件(启用自动上传的功能)：<br/>

	//初始化uploader
	var uploader = WebUploader.create({
		auto: true,
    	swf : '../statics/plugins/webuploader/Uploader.swf',
    	server : '../selfassessment/uploadfile',
    	pick : {
    		"id": '#picker',
    		"multiple":false
    	},
    	fileVal : 'file', //提交到到后台，是要用"multiFile"这个名称属性来取文件的
    	fileNumLimit:5,//上传的文件总数 (我理解的意思是单次不刷新页面能上传的次数)
    	fileSingleSizeLimit: 2*1024*1024,
    	duplicate :true,
    	accept: {
    	    title: 'Files',
    	    extensions: 'gif,jpg,jpeg,bmp,png,doc,docx,pdf,txt,zip,rar,xlsx,ppt,pptx'/*,
    	    mimeTypes: 'file/*'*/
    	},
    	formData: {
    		uid: 123
    	},
        
        uploadSuccess : function(file) {
            $('#' + file.id).find('p.state').text('已上传');
        },
        uploadError : function(file) {
            $('#' + file.id).find('p.state').text('上传出错');
        },
        uploadComplete : function(file) {
            $('#' + file.id).find('.progress').fadeOut();
        }
    });
	
	//uploader.options.formData.uid = $("selfAssessmentId").val();
	
	uploader.on( 'uploadBeforeSend', function( block, data ) {
	    // 修改data可以控制发送哪些携带数据。
	    data.uid = $('#selfAssessmentId').val();
	});
	
	uploader.on( 'fileQueued', function( file ) {
		var $list = $('#thelist');
	    $list.append( '<div id="' + file.id + '" class="item">' +
	        '<h4 class="info">' + file.name + '</h4>' +
	        '<p class="state">等待上传...</p>' +
	    '</div>' );
	});
	
	uploader.on("error", function (type) {
	    if (type == "Q_TYPE_DENIED") {
	    	alert("上传的格式不正确,支持上传 gif, jpg, jpeg, bmp, png, doc, docx, pdf, txt, zip, rar, xlsx, ppt, pptx");
	    } else if (type == "Q_EXCEED_SIZE_LIMIT") {
	        alert("文件大小不能超过2M");
	    } else if(type == "F_EXCEED_SIZE") {
	    	alert("文件过大,文件大小不能超过2M")
	    } else if(type== "Q_EXCEED_NUM_LIMIT") {
	    	alert("警告：您频繁上传且频繁删除附件已被系统检测，请规范你的操作！去刷新页面试试");
	    }
	    else {
	    	alert("上传出错！请检查后重新上传！错误代码"+type);
	    }
	});
	
	uploader.on( 'uploadProgress', function( file, percentage ) {
	    var $li = $( '#'+file.id ),
	    $percent = $li.find('.progress .progress-bar');
	    // 避免重复创建
	    if ( !$percent.length ) {
	        $percent = $('<div class="progress progress-striped active">' +
	          '<div class="progress-bar" role="progressbar" style="width: 0%">' +
	          '</div>' +
	        '</div>').appendTo( $li ).find('.progress-bar');
	    }
	    $li.find('p.state').text('上传中');
	    $percent.css( 'width', percentage * 100 + '%' );
	});
	
	uploader.on( 'uploadSuccess', function( file ) {
	    $( '#'+file.id ).find('p.state').text('已上传');
		//这里是我自己的页面逻辑
	    $('.btns').css('display','none');
	    $('#removefile').css('display','inline');
	});
	
	uploader.on( 'uploadError', function( file ) {
	    $( '#'+file.id ).find('p.state').text('上传出错');
	});
	
	uploader.on( 'uploadComplete', function( file ) {
	    $( '#'+file.id ).find('.progress').fadeOut();
	    return;
	});
    
    $('#ctlBtn').on('click', function() {
    	console.log("上传...");
        uploader.upload();
        console.log("上传成功");
    });
&nbsp;&nbsp;&nbsp;&nbsp;4、页面代码处理：<br/>

	<div id="uploader" class="wu-example">
	 <!--用来存放文件信息-->
	    <div id="thelist" class="uploader-list"></div>
	    <div class="btns">
	        <div id="picker" data-toggle="tooltip" data-placement="top" title="选择好文件后即立即上传">选择附件</div>
	        <!-- <button id="ctlBtn" class="btn btn-default">开始上传</button> -->
	    </div>
	</div>
&nbsp;&nbsp;&nbsp;&nbsp;5、mvc 后台处理：<br/>

	@RequestMapping(value="/uploadfile")
	public void uploadfile(@RequestParam("file") MultipartFile file,HttpServletRequest request) throws Exception{
        FileUtil util = new FileUtil();
        String originFilename = file.getOriginalFilename().substring(0,file.getOriginalFilename().lastIndexOf("."));
        String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf(".") + 1);
        String realPath = "D:\\fileUpload\\";
        File destFile = new File(realPath);
        if (!destFile.exists()) {
            destFile.mkdirs();
        }
        String fileNameNew = util.getFileNameNew() + "." + suffix;
        File f = new File(destFile.getAbsoluteFile() + "/" + fileNameNew);
        file.transferTo(f);
        f.createNewFile();
		//AttachmentEntity该类是数据库存储附件信息的对应的基类
        AttachmentEntity attachmentEntity = new AttachmentEntity();
        attachmentEntity.setFileName(originFilename+"."+suffix);
        attachmentEntity.setFileUrl("D:\\fileUpload\\"+fileNameNew);
        attachmentEntity.setExpandedName(fileNameNew);
        attachmentEntity.setSuffix(suffix);
        attachmentEntity.setBusinessId(Integer.parseInt(request.getParameter("uid")));
        attachmentService.save(attachmentEntity);
	}

--------
	工具类：
	public class FileUtil {
	    public String getFileNameNew() {
	    	SysUserEntity sysUserEntity = ShiroUtils.getUserEntity();
	        SimpleDateFormat fmt = new SimpleDateFormat("yyyyMMddHHmmssSSS");
	        String returnStr = sysUserEntity.getEmployId()+"_"+fmt.format(new Date());
	        return returnStr;
	    }
	}

--------
#### 2、文件下载功能	
&nbsp;&nbsp;&nbsp;&nbsp;1、HTML代码：</br>

	<input type="button" class="btn btn-primary" click="downloadFile" value="下载" />
&nbsp;&nbsp;&nbsp;&nbsp;2、js代码<br/>

	downloadFile: function(event) {
		location.href = "../scoregroup/downloading?expandedName=" + vm.selfAssessment.expandedName+'&fileName='+vm.selfAssessment.fileName;
	}
&nbsp;&nbsp;&nbsp;&nbsp;3、这里提供两种下载方式，建议使用第二种<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方式一：

	@RequestMapping(value = "/download")
	public ResponseEntity<byte[]> download(@RequestParam(name="expandedName")String expandedName, @RequestParam(name="fileName")String fileName,
	        HttpServletRequest request) {
	   HttpHeaders headers = new HttpHeaders();
        try {
            headers.setContentDispositionFormData("attachment", fileName);
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            // 获取物理路径
            File file = new File("D:\\fileUpload\\"+expandedName);
            return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file), headers, HttpStatus.CREATED);
        } catch (Exception e) {
            e.printStackTrace();
        }
	     
	    return null;
	}

--------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方式二：

	@RequestMapping(value = "/downloading")
	public void downloading(@RequestParam(name="expandedName")String expandedName, @RequestParam(name="fileName")String fileName,HttpServletResponse response) throws Exception {
        
		File file=new File("D:\\fileUpload\\"+expandedName);
		InputStream in=new FileInputStream(file);
		response.setCharacterEncoding("utf-8");
		response.setContentType("multipart/form-data");
		response.setHeader("Content-Disposition","attachment;fileName="+new String(fileName.getBytes("gbk"),"iso-8859-1"));//避免文件名乱码

		OutputStream out = null;
		try {
			out = response.getOutputStream();
		} catch (IOException e) {
			e.printStackTrace();
		}
		int b=0;
		while((b=in.read())!=-1)
			out.write(b);

		out.flush();
		in.close();
		out.close();
	}


