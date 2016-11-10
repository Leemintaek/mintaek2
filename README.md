# mintaek2
////////////////////////////////////////////////////////////////////////////////////////////
/// Attributes
////////////////////////////////////////////////////////////////////////////////////////////


////////////////////////////////////////////////////////////////////////////////////////////
/// Operations
////////////////////////////////////////////////////////////////////////////////////////////

function initAnnouncementHome() {
	setTitle("알림", true);
	resetContent("announcement_content");	
	
	var view = '';
	
	view += '<div class="edit_wrap announcement_edit_wrap"></div>';
	view += '<div class="list_section"></div>';
	
	$("#content").append(view);
	
	setAnnouncementEditView($("#content .announcement_edit_wrap"), "write", null);
	
	requestAnnouncementList();	
}

function refreshAnnouncementListView() {
	$("form[name=announcement_write_form]").each(function() {
		this.reset();
	});
	
	requestAnnouncementList();
}

function uploadAnnouncement(target, queue, id) {	
	target.uploadify("settings", "formData", { "id": id });	
	target.uploadify("upload", "*");
	
	if(queue.children().length == 0) {
		refreshAnnouncementListView();
	}
}

function setAnnouncementListView(announcements) {
	var view = '';
	
	$(announcements).each(function(index, announcement) {
		view += '<div class="item">';
		view += 	'<div class="item_body">';
		view += 		'<a class="announcement_item" href="' + announcement.id + '">';
		view +=			'<div class="substance_announcement menu">' + announcement.overview + '</div>';
		view +=			'<div class="menu devide25_mobile">' + announcement.type + '</div>';
		view +=			'<div class="menu devide25_mobile">' + announcement.language + '</div>';
		view +=			'<div class="menu devide25_mobile">' + announcement.status + '</div>';
		view +=			'<div class="menu devide25_mobile">' + announcement.exposedCount + '(' + announcement.clickedCount + ')</div>';
		view +=		'</a>';
		view +=	'</div>';
		view += '</div>';
	});	
		
	$("#content .list_section").empty();
	$("#content .list_section").append(view);
	
	$("#announcement_write_upload").trigger("click");		// 왜 이곳에 해야지만 먹힐까..ㅡ,ㅡ
}

function setAnnouncementEditView(target, type, data) {	
	var formName = '';
	var uploadName = '';
	var submitBtn = '';
	var appendix = '';
	
	var id = '-1';
	
	if(type == "write") {
		formName = "announcement_write_form";
		uploadName = "announcement_write_upload";
		submitBtn = '올리기';		
	}		
	else {
		formName = "announcement_modify_form";
		uploadName = "announcement_modify_upload";
		submitBtn = '수정하기';
		id = data.id;
		
		appendix += '<div class="overflow">';
		appendix += 	'<span class="devide5_mobile">작성자 : ' + data.writerId + '</span>';		
		appendix +=		'<span class="devide5_mobile" title="' + data.writtenDateTime + '">작성일 : ' + data.writtenDateTime + '</span>';		
		appendix +=		'<span class="devide5_mobile">수정자 : ' + data.lastModifierId + '</span>';
		appendix +=		'<span class="devide5_mobile" title="' + data.lastModifiedDateTime + '">수정일 : ' + data.lastModifiedDateTime + '</span>';		
		appendix +=	 '</div>';
	}
	
	var view = '';
	
	view += '<form name="' + formName + '" data-id="' + id + '">';
	view += 	appendix;
	view += 	'<input type="text" name="overview" placeholder="개요를 입력하세요"/>';
	view += 	'<input type="text" name="title" placeholder="제목을 입력하세요"/>';
	view += 	'<textarea name="description" placeholder="내용을 입력하세요"></textarea>';
	view += 	'<input type="text" name="link" placeholder="링크를 입력하세요"/>';	
	view += 	'<div class="select">';
	view += 		'<select name="type">';
	view += 			'<option value="">유형</option>';
	view += 			'<option value="Notice">알림</option>';
	view += 			'<option value="Information">정보</option>';
	view += 			'<option value="Event">이벤트</option>';
	view += 		'</select>';
	view += 	'</div>';				
	view += 	'<div class="select">';
	view += 		'<select name="imageName">';
	view += 			'<option value="">이미지</option>';	
	view += 			'<option value="img_system_check.png">img_system_check.png</option>';
	view += 			'<option value="img_sorry.png">img_sorry.png</option>';
	view += 			'<option value="img_event.png">img_event.png</option>';
	view += 		'</select>';
	view += 	'</div>';	
	view += 	'<div class="select">';
	view += 		'<select name="language">';
	view += 			'<option value="">언어</option>';
	view += 			'<option value="ko">한국어</option>';
	view += 			'<option value="en">영어</option>';
	view += 			'<option value="ja">일본어</option>';
	view += 			'<option value="zh">중국어</option>';
	view += 			'<option value="en,ja">영어/일본어</option>';
	view +=			'<option value="en,zh">영어/중국어</option>';
	view += 			'<option value="en,ja,zh">영어/일본어/중국어</option>';
	view += 		'</select>';
	view += 	'</div>';	
	view += 	'<div class="select">';
	view += 		'<select name="status">';
	view += 			'<option value="">상태</option>';
	view += 			'<option value="Active">활성</option>';
	view += 			'<option value="Deactive">비활성</option>';			    			
	view += 		'</select>';
	view += 	'</div>';
	view += 	'<div class="overflow">';	
	view += 		'<input type="file" name="files" id="' + uploadName + '"/>';
	view +=		'<div class="upload_files"></div>';
	view += 		'<div class="picture_right">';								
	view += 			'<div class="table_right_picture">';
	view += 				'<input type="submit" class="ok" value="' + submitBtn + '"/>';
	view += 			'</div>';	
	view += 		'</div>';
	view += 	'</div>';						
	view += '</form>';	
	
	if(type == "modify") {
		$(".list_section .announcement_edit_wrap").prev().removeClass("bold");
		$(".list_section .announcement_edit_wrap").remove();
		
		view = '<div class="edit_wrap announcement_edit_wrap">' + view + '</div>';
	}		
	
	target.append(view);
	
	if(type == "write")
		return;
	
	// modify...
	target.find(".item_body").addClass("bold");
	
	target.find("input[name=overview]").val(data.overview);
	target.find("input[name=title]").val(data.title);
	target.find("textarea[name=description]").val(data.description);	
	target.find("input[name=link]").val(data.link);	
	target.find("select[name=type]").val(data.type);
	target.find("select[name=imageName]").val(data.imageName);
	target.find("select[name=language]").val(data.language);
	target.find("select[name=status]").val(data.status);
	
	if(data.fileList == null)
		return;
	
	view = '<ul>';
	
	$(data.fileList).each(function(index, file) {
		view += '<li>';
		view += 	'<span class="block">';
		view += 		'<span class="image"><img src="' + file.thumbnailLocation + '"/></span>';	
		view += 		'<button name="announcement_remove_file_btn" data-id="' + data.id + '" data-fileNumber="' + file.itemNumber + '">';
		view += 			'<img src="../_base/images/btn_close.png">';
		view += 		'</button>';
		view += 	'</span>';											
		view += '</li>';		
	});
	
	view += '</ul>';
	
	target.find(".upload_files").append(view);	
	
	$("#announcement_modify_upload").trigger("click");
}


////////////////////////////////////////////////////////////////////////////////////////////
/// Event Handlers
////////////////////////////////////////////////////////////////////////////////////////////

$(document).ready(function() {	
	$(document).on("submit", "form[name=announcement_write_form]", function(event) {
		if(checkAnnouncement($("form[name=announcement_write_form]")))
			requestAnnouncementWrite($(this).serialize());			
		
		return false;
	});
	
	$(document).on("submit", "form[name=announcement_modify_form]", function(event) {
		if(checkAnnouncement($("form[name=announcement_modify_form]")))
			requestAnnouncementModify($(this).attr("data-id"), $(this).serialize());			
		
		return false;
	});
	
	$(document).on("click", ".announcement_item", function(event) {
		requestAnnouncementDetail($(this).attr("href"), $(this).parent().parent());
		return false;
	});
	
	$(document).on("click", "button[name=announcement_remove_file_btn]", function(event) {		
		var target = $(this).parent().parent();
		var id = $(this).attr("data-id");
		var fileNumber = $(this).attr("data-fileNumber");
		
		setAlertMessage("선택한 이미지를 삭제하시겠습니까?");
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: "예",
	            	click: function() {
	            		$(this).dialog("close");
	            		requestAnnouncementRemoveFile(id, fileNumber);
	            		target.remove();
	            	}
	            },	            
	            {
	            	text: "아니오",
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]			
		});	
		
		return false;
	});		
	
	$(document).on("click", "#announcement_write_upload", function(event) {
		$(this).uploadify({
			method: "post",
		    fileTypeDesc: "이미지파일",
		    fileTypeExts: "*.jpg; *.jpeg; *.gif; *.png",
		    buttonImage: "images/btn_picture.png",		    		    
		    width: 25,
		    height: 25,
		    swf: "../_base/uploadify/uploadify.swf",
	        uploader: "/announcement/upload_files.dive",
	        multi: true,
	        auto: false,
	        onQueueComplete: function(queueData) {	        	
	        	refreshAnnouncementListView();
	        }
		});
		
		return false;
	});
	
	$(document).on("click", "#announcement_modify_upload", function(event) {
		$(this).uploadify({
			method: "post",
		    fileTypeDesc: "이미지파일",
		    fileTypeExts: "*.jpg; *.jpeg; *.gif; *.png",
		    buttonImage: "images/btn_picture.png",		    		    
		    width: 25,
		    height: 25,
		    swf: "../_base/uploadify/uploadify.swf",
	        uploader: "/announcement/upload_files.dive",
	        multi: true,
	        auto: false,
	        onQueueComplete: function(queueData) {	        	
	        	refreshAnnouncementListView();
	        }
		});
		
		return false;
	});
});


////////////////////////////////////////////////////////////////////////////////////////////
/// Request Data
////////////////////////////////////////////////////////////////////////////////////////////

function requestAnnouncementList() {
	$.ajax({
	    type: "GET",
	    url: "/announcement/list.dive",	    
	    dataType: "json",
	    cache: false,
	    success: function(resultMap) {
	    	if(isSucceeded(resultMap))
	    		setAnnouncementListView(resultMap.data);
	        else
	        	showResultMessage(resultMap);
	    },
		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
		beforeSend: function() { showProgress(); },
		complete: function() { hideProgress(); }
	});		
}

function requestAnnouncementDetail(id, target) {
	$.ajax({
	    type: "GET",
	    url: "/announcement/detail.dive?id=" + id,
	    dataType: "json",
	    cache: false,
	    success: function(resultMap) {
	    	if(isSucceeded(resultMap))
	    		setAnnouncementEditView(target, "modify", resultMap.data);
	        else
	        	showResultMessage(resultMap);
	    },
		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
		beforeSend: function() { showProgress(); },
		complete: function() { hideProgress(); }
	});	
}

function requestAnnouncementWrite(request_data) {	
	$.ajax({
	    type: "POST",
	    url: "/announcement/write.dive",
	    data: request_data,
	    dataType: "json",
	    cache: false,
	    success: function(resultMap) {
	    	if(isSucceeded(resultMap))
	    		uploadAnnouncement($("#announcement_write_upload"), $("#announcement_write_upload-queue"), resultMap.data);
	        else	        	
	            showResultMessage(resultMap);
	    },
		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
		beforeSend: function() { showProgress(); },
		complete: function() { hideProgress(); }
	});
}

function requestAnnouncementModify(id, request_data) {
	$.ajax({
  		type: "POST",
  		url: "/announcement/modify.dive?id=" + id,
  		data: request_data,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				uploadAnnouncement($("#announcement_modify_upload"), $("#announcement_modify_upload-queue"), resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestAnnouncementRemoveFile(id, fileNumber) {
	$.ajax({
	    type: "GET",
	    url: "/announcement/remove_file.dive?announcementId=" + id + "&fileNumber=" + fileNumber,
	    dataType: "json",
	    cache: false,
	    success: function(resultMap) {
	    	if(isError(resultMap))
  				showResultMessage(resultMap);
	    },
		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
		beforeSend: function() { showProgress(); },
		complete: function() { hideProgress(); }
	});		
}


////////////////////////////////////////////////////////////////////////////////////////////
/// Check Validation
////////////////////////////////////////////////////////////////////////////////////////////

function checkAnnouncement(form) {
	var overview = form.find("input[name=overview]").val();
	
	if(overview.length == 0) {
		showMessage("개요를 입력해주십시오."); 
		return false;
	}
	
	var title = form.find("input[name=title]").val();
	
	if(title.length == 0) {
		showMessage("제목을 입력해주십시오."); 
		return false;
	}
	
	var description = form.find("textarea[name=description]").val();
	
	if(description.length == 0) {
		showMessage("내용을 입력해주십시오."); 
		return false;
	}
	
	var link = form.find("input[name=link]").val();
	
	if(link.length > 0) {
		if(link.length > 100) {
			showMessage("링크를 100자이내로 입력해주십시오.");
			return false;
		}
		
		var expLink = /^([a-z]+):\/\/((?:[a-z가-힣\d\-_]{1,}\.)+[a-z]{1,})(:\d{1,5})?(\/[^\?]*)?(\?.+)?$/i;
		
		if(!expLink.test(link)) {
			expLink = /^((?:[a-z가-힣\d\-_]{1,}\.)+[a-z]{1,})(:\d{1,5})?(\/[^\?]*)?(\?.+)?$/i;
			
			if(!expLink.test(link)) {
				showMessage("URL 형식이 옳바르지 않습니다.");
				return false;
			}			
		}			
	}
	
	var type = form.find("select[name=type]").val();
	
	if(type == "") {
		showMessage("유형을 선택해주십시오.");
		return false;
	}
	
	var language = form.find("select[name=language]").val();
	
	if(language == "") {
		showMessage("언어를 선택해주십시오.");
		return false;
	}
	
	var status = form.find("select[name=status]").val();
	
	if(status == "") {
		showMessage("상태를 선택해주십시오.");
		return false;
	}
	
	return true;
}
