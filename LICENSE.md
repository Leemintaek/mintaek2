////////////////////////////////////////////////////////////////////////////////////////////
/// Attributes
////////////////////////////////////////////////////////////////////////////////////////////

var g_logbookPageSize = 20;
var g_logbookCurrentPageNumber = 1;
var g_logbookTotalPageCount = 0;
var g_logbookSelectedMenu = "";

var g_logbookBestCount = 5;
var g_logbookGalleryView = "";
var g_logbookStickerView = "";
var g_logbookSelectedSticker = "";

var g_logbookSelectedUserId = '';
var g_logbookSelectedLogbookNumber = '';
var g_logbookSelectedItem = null;

var g_logbookIsAd = false;
var g_logbookPrevAd = false;
var g_logbookIsPostedAd = false;

var g_logbookGraph = null;
var g_logbookDetailGraph = null;

var g_logbookImportMaxCount = 20;
var g_logbookImportTotalCount = 0;
var g_logbookImportedCount = 0;


////////////////////////////////////////////////////////////////////////////////////////////
/// Operations
////////////////////////////////////////////////////////////////////////////////////////////

function initLogbookHome() {
	g_logbookPageSize = 20;
	g_logbookCurrentPageNumber = 1;
	g_logbookTotalPageCount = 0;
	
	handleLogbookMenu("logbook_all_list_view");
}

function handleLogbookMenu(menu) {
	clearContents();
	$("#contents").append('<div id="log_book"></div>');
	
	g_logbookSelectedMenu = menu;
	
	if(menu == "logbook_write_view")
    	initLogbookWriteView();
	else if(menu == "logbook_bulk_import_view")
		initLogbookBulkImportView();
	else
		initLogbookListView();
}

function initLogbookListView() {
	var category = '';
	
	category += '<li><a href="logbook_all_list_view" class="active">' + lang["log_all"] + '</a></li>';
	
	if(isMyHome()) {
		category +=	'<li><a href="logbook_signed_list_view">' + lang["log_signed"] + '</a></li>';
		category +=	'<li><a href="logbook_statistics_view">' + lang["log_statistics"] + '</a></li>';
	}		
	
	var view = '';
	
	view += '<h1>' + lang["logbook"] + '</h1>';
	view += '<div class="category">';	
	view += 	'<ul>';
	view += 		category;
	view +=	'</ul>';
	view += '</div>';
	view += '<div id="news_feed" class="log_book_list">';	
	view += 	'<div class="post_section"></div>';
	view += 	'<div class="post_pager"></div>';
	view += 	'<div class="post_search">';
	view +=		'<form name="logbook_search_form">';
	view +=			'<div class="selector">';
	view += 				'<select name="logbookSearchType">';
	view += 					'<option value="Location" selected>' + lang["location"] + '</option>';
	view += 					'<option value="Point">' + lang["point"] + '</option>';
	view += 					'<option value="Activity">' + lang["activity"] + '</option>'; 
	view += 					'<option value="DiveNumber">' + lang["dive_number_full"] + '</option>';
	view += 					'<option value="DiveYear">' + lang["date_year"] + '</option>';
	view += 					'<option value="DiveMonth">' + lang["date_month"] + '</option>';
	view += 				'</select>';
	view += 			'</div>&nbsp;';
	view += 			'<div class="keyword">';	
	view += 				'<input type="text" name="condition"/>';
	view += 				'<input type="image" src="./main/images/btn_keyword_search.png" />';	
	view += 			'</div>';
	view +=		'</form>';
	view +=	'</div>';
	view += '</div>';
	view += '<div id="statistics" class="log_book_list">';
	view += '</div>';
	
	
	$("#log_book").append(view);
	
	changeLogbookCategory("logbook_all_list_view");
}

function initLogbookStatisticsPerMonthView(startYear) {
	var date = new Date();
	var currentYear = date.getFullYear();

	var target = $("#log_month select");
	var view = '';
	
	for(var i=startYear; i<=currentYear; i++) {
		if(i == currentYear)
			view += '<option value="' + i + '" selected>' + i + '</option>';
		else
			view += '<option value="' + i + '">' + i + '</option>';
	}
	
	$("#log_month select").empty();
	$("#log_month select").append(view);
	
	requestLogbookStatisticsPerMonth(g_userId, currentYear);
}

function initLogbookWriteView() {	
	/*
	// set focus logbook menu...
	$(".left_menu").each(function(index, item) {
		$(item).find("a").removeClass("current");
	});
	
	$(".left_menu").find("a[href=logbook_home]").addClass("current");
	*/	
	
	requestLogbookWriteForm();
}

function initLogbookBulkImportView() {
	var view = '';
	
	view += '<h1>' + lang["import_log_files"] + '</h1>';	
	view += '<div class="upload_bulk_information">';
	view += 	'<div class="upload_image">';
	view +=		'<img src="./main/images/upload.png">';
	view +=	'</div>';
	view +=	'<div class="upload_info">';
	view +=		'<ul>';
	view +=			'<li>' + lang["upload_desc1"] + '&nbsp;&nbsp;&nbsp;&nbsp;<a class="logbook_help_import_btn" href="#">[' + lang["help"] + ']</a></li>';
	view +=			'<li>' + lang["upload_desc2"] + '</li>';
	view +=			'<li>' + lang["upload_desc3"] + '</li>';
	view +=		'</ul>';
	view +=	'</div>';
	view += '</div>';
	view += '<hr>';
	view += '<div class="upload_files">';
	view += 	'<div class="buttons">';
	view +=		'<button name="logbook_bulk_upload_btn">' + lang["upload"] + '</button>';
	view +=		'<button name="logbook_sort_btn">' + lang["sort_logs"] + '</button>';
	view +=		'<div class="filebox">';
	view +=		'</div>';
	view +=	'</div>';
	view +=	'<div class="list">';
	view +=		'<ul id="logbook_bulk_import_list">';
	view +=		'</ul>';
	view +=	'</div>';
	view +=	'<div class="buttons">';
	view +=		'<button name="logbook_bulk_upload_btn">' + lang["upload"] + '</button>';
	view +=	'</div>';
	view += '</div>';	
	
	$("#log_book").append(view);	
	
	initLogbookBulkImportChooseFilesView();
}

function initLogbookBulkImportChooseFilesView() {
	var view = '';
	
	view += '<label for="log_files">' + lang["choose_files"] + '</label>';
	view += '<input type="file" id="log_files" multiple="multiple" onchange="initLogbookBulkImportFilesView();">';
	
	$("#log_book .filebox").empty();
	$("#log_book .filebox").append(view);
	
	$("#log_book").find(".filebox").show();
	$("#log_book").find("button[name=logbook_sort_btn]").show();
	$("#log_book").find("button[name=logbook_bulk_upload_btn]").hide();	
}

function initLogbookBulkImportFilesView() {
	var files = document.getElementById("log_files").files;
	
	if(getSelectedImportFileCount(files) > g_logbookImportMaxCount) {
		showMessage(lang["exceed_import_files"]);
		initLogbookBulkImportChooseFilesView();
		return;
	}
		
	var view = '';
	
	$(files).each(function(index, file) {		
		view += '<li id="logbook_import_file_' + index + '" data-canceled="false">';
		view += 	'<span class="left">';
		view +=		'<span class="ellipsis" title="' + file.name + '">' + file.name + '</span>';
		view +=			'<span class="uploading_btn"><img src="./_base/images/uploading_file.gif"></span>';
		view +=			'<button class="cancel_btn" data-index="' + index + '"><img src="./main/images/btn_progress_close.png"></button>';
		view +=		'</span>';
		view +=		'<span class="border"/>';	
		view +=	'<span class="right ellipsis">' + lang["wait"] + '</span>';
		view += '</li>';
	});
	
	$("#logbook_bulk_import_list").empty();
	$("#logbook_bulk_import_list").append(view);
	
	$("#log_book").find("button[name=logbook_bulk_upload_btn]").show();
	$("#log_book").find("button[name=logbook_sort_btn]").hide();
	$("#log_book").find(".filebox").hide();		
	
	g_logbookImportTotalCount = getSelectedImportFileCount(document.getElementById("log_files").files);
	g_logbookImportedCount = 0;
}

function readLogFiles(type, index, command) {
	var files = document.getElementById("log_files").files;	
	var count = type == "Bulk" ? getSelectedImportFileCount(files) : files.length;
	
	if(count == 0) {
		showMessage(lang["logbook_import_select_file"]);
		return;
	}	
		
	if(files.length < (index + 1))
		return;		
	
	if(type == "Bulk") {
		var item = $("#logbook_import_file_" + index);
		
		if(item.attr("data-canceled") == "true")
			return readLogFiles(type, ++index, command);
		
		item.find(".cancel_btn").hide();
		item.find(".uploading_btn").show();
		
		item.find(".right").empty();
		item.find(".right").append(lang["uploading"]);
		item.find(".right").attr("title", lang["uploading"]);
	}
	
	var file = files[index];
	var format = getFileExtension(file.name);	
	var reader = new FileReader();
	
	reader.onload = function(event) {
		requestLogbookImport(type, index, format, event.target.result, command);
	};
	
	reader.onerror = function(event) {
		var errCode = event.target.error.code;
		var message = ""; 
		
		if(errCode == 1)
			message = lang["logbook_import_file_not_found"];
		else if(errcode == 2)
			message = lang["logbook_import_file_unsafe"];
		else if(errcode == 3)
			message = lang["logbook_import_file_stop_read"];
		else if(errcode == 4)
			message = lang["logbook_import_file_security"];
		else if(errcode == 5)
			message = lang["logbook_import_file_url"];
		
		if(type == "Bulk")
			setLogbookBulkImportResult(index, false, message, null);
		else
			showMessage(message);
	};
	
	reader.readAsText(file);		// 기본 인코딩 : utf-8	
}

function initLogbookCopiedWriteView(userId, logbookNumber) {
	clearContents();
	$("#contents").append('<div id="log_book"></div>');
	
	g_logbookSelectedMenu = "logbook_write_view";
		
	requestLogbookCopiedWriteForm(userId, logbookNumber);
}

function initLogbookModifyView(dive) {
	clearContents();
	$("#contents").append('<div id="log_book"></div>');
	
	g_logbookSelectedMenu = "logbook_modify_view";
	
	setLogbookEditView('modify', dive);
	
	if(dive.signatureStatus == "Put") {
		setAlertMessage(lang["edit_restricted_logbook"]);
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: lang["confirm"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]		
		});
	}
}

function changeLogbookCategory(menu) {
	g_logbookSelectedMenu = menu;	
	g_logbookCurrentPageNumber = 1;
	
	refreshLogbookListView(g_logbookCurrentPageNumber);
}

function refreshLogbookListView(pageNumber) {
	if(g_mainSelectedMenu == "news") {
		refreshNewsListView();
		return;		
	}
	
	if(g_logbookSelectedMenu == "logbook_write_view" || g_logbookSelectedMenu == "logbook_modify_view") {
		handleLogbookMenu("logbook_list_view");
		return;
	}	
	
	if(g_mainSelectedMenu == "search") {
		requestSearchLogbookList($("input[name=integrated_condition]").val(), pageNumber);
		requestSearchLogbookCount($("input[name=integrated_condition]").val());
	}
	else {		
		if(g_logbookSelectedMenu == 'logbook_statistics_view') {
			$("#statistics").show();
			$("#statistics").empty();
			
			$("#news_feed").hide();
		}
		else {
			$("#news_feed").show();
			$("#news_feed .post_section").empty();
			$("#news_feed .post_pager").empty();
			
			$("#statistics").hide();
		}
		
		if(g_logbookSelectedMenu == 'logbook_all_list_view') {
			requestLogbookList(g_userId, $("form[name=logbook_search_form]").serialize(), g_logbookCurrentPageNumber);
			requestLogbookCount(g_userId, $("form[name=logbook_search_form]").serialize());
		}
		else if(g_logbookSelectedMenu == 'logbook_signed_list_view') {
			requestLogbookSignedList(g_userId, $("form[name=logbook_search_form]").serialize(), g_logbookCurrentPageNumber);
			requestLogbookSignedCount(g_userId, $("form[name=logbook_search_form]").serialize());
		}
		else if(g_logbookSelectedMenu == 'logbook_statistics_view')
			requestLogbookStatisticsOverview(g_userId);
	}
}

function uploadLogbook(mode, userId, logbookNumber, pageNumber) {
	g_logbookSelectedUserId = userId;
	g_logbookSelectedLogbookNumber = logbookNumber;
	
	if(mode == "write")
		g_logbookIsAd = $("form[name=logbook_write_form] input[name=isAd]").prop("checked");
	else if(mode == "modify")
		g_logbookIsAd = $("form[name=logbook_modify_form] input[name=isAd]").prop("checked");
	else if(mode == "modify_restricted")
		g_logbookIsAd = $("form[name=logbook_modify_restricted_form] input[name=isAd]").prop("checked");

	$("#upload_logbook").uploadify("settings", "formData", { "userId": userId, "logbookNumber": logbookNumber });	
	$("#upload_logbook").uploadify("upload", "*");
	
	if($("#upload_logbook-queue").children().length == 0) {
		hidePopup();		
		exportLogbookAd();
		refreshLogbookListView(pageNumber);
	}		
}

function exportLogbookAd() {
	if(g_myType != "Sponsor")
		return;
	
	if(g_logbookIsAd == g_logbookPrevAd)
		return;
	
	requestLogbookAd(g_logbookSelectedUserId, g_logbookSelectedLogbookNumber, g_logbookIsAd);
}

function exportLogbook(target, dive) {
	// name...
	var name = dive.location;
	
	if(dive.point.length > 0)
		name += ' - ' + dive.point;		
	
	// description...
	var description = '';
	
	description += lang["dive_number_full"] + ' : ' + dive.diveNumber + ', ';
	description += lang["dive_date"] + ' : ' + dive.diveDate + ', ';	
	description += lang["max_depth"] + ' : ' + dive.maxDepth + getDistanceUnit(dive.distanceUnit);
	description += lang["average_depth"] + ' : ' + dive.averageDepth + getDistanceUnit(dive.distanceUnit);
	description += lang["dive_time"] + ' : ' + dive.diveTime + ' min, ';
	description += lang["safety_stop_time"] + ' : ' + dive.safetyStopTime + ' min, ';
	
	description += lang["visibility"] + ' : ' + dive.visibility + getDistanceUnit(dive.distanceUnit);
	description += lang["min_temperature"] + ' : ' + dive.minTemperature + getTemperatureUnit(dive.temperatureUnit);
	
	if(dive.activity.length > 0)
		description += ', ' + lang["activity"] + ' : ' + dive.activity;
	
	// link...
	//var link = 'http://www.divememory.com/contents.jsp?appType=' + appType + '&itemType=logbook&itemId=' + dive.userId + '&itemNumber=' + dive.logbookNumber;
	//var picture = 'http://www.divememory.com' + getExportImage(dive.fileList);
	
	var link = makeLink("logbook", dive.userId, dive.logbookNumber);
	var picture = getExportImage(dive.fileList);		
	
	// feed...
	var feed = {
		method: 'feed',
		name: name,		
		caption: 'www.divememory.com',
		description: description,			
		link: link,
		picture: picture
	};
	
	FB.ui(feed, function(response) {
		if(response && response.post_id)
			showMessage(lang["share_logbook_success"]);
		else
			showMessage(lang["share_logbook_failure"]);
		
		requestLogbookShare(target, dive.userId, dive.logbookNumber);
	});
}

function getLogbookId(userId, logbookNumber) {
	type = isVisiblePopup() ? 'logbook_detail__' : 'logbook__'; 
	return  type + userId + '__' + logbookNumber;
}

function getLogbookStickerId(id) {
	return 'logbook_sticker_' + id;
}

function getLogbookStickerObject(id) {
	return $('#' + getLogbookStickerId(id));
}

function getLogbookObject(userId, logbookNumber) {
	return $('#' + getLogbookId(userId, logbookNumber));
}

function getSelectedImportFileCount(files) {
	var count = 0;
	
	$(files).each(function(index, file) {
		var item = $("#logbook_import_file_" + index);
		
		if(item.attr("data-canceled") == "false")
			count++;
	});
	
	return count;
}

function autoTimeOut() {
	var timeInHour = $(document).find("input[name=timeInHour]").val();
	var timeInMinute = $(document).find("input[name=timeInMinute]").val();
	var diveTime = $(document).find("input[name=diveTime]").val();
	
	if(timeInHour == null || timeInMinute == null || diveTime == null)
		return;
	
	if(timeInHour.length == 0 || timeInMinute.length == 0 || diveTime.length == 0)
		return;
	
	var timeOutHour = parseInt(timeInHour) || 0;
	var timeOutMinute = (parseInt(timeInMinute) || 0) + (parseInt(diveTime) || 0);
	
	if(timeOutMinute == 0)
		return;
	
	timeOutHour += parseInt(timeOutMinute / 60);
	timeOutMinute = timeOutMinute % 60;
	
	$(document).find("input[name=timeOutHour]").val(timeOutHour);
	$(document).find("input[name=timeOutMinute]").val(timeOutMinute);
}

function generateLogbookStickers() {
	var stickers = '';
	
	$(".sticker_list ul li").each(function() {
		stickers += $(this).find("button").attr("name").split("_")[2] + ";";
	});
	
	$("input[name=stickers]").val(stickers);
}

function addLogbookVideoLink(link) {	
	link = link.replace(/"/gi, "'");
	
	var view = '';
	
	view += '<li>';
	view += 	'<div class="input_uri"><input type="text" name="links" readonly="readonly" value="' + link + '" /></div>';
	view += 	'<div class="cancel"><button type="button" name="logbook_remove_video_link_btn"><img src="./main/images/btn_progress_close.png" /></button></div>';
	view += '</li>';
	
	$(".logbook_video_link").show();	
	$(".logbook_video_link ul").append(view);
}

function parseLogbookProfile(dive) {
	var profile = {};
	
	var labels = '';
    var depth = '';
    var temperature = '';
    var currentTemperature = 0;
    
    if(dive.inputType == "Device") {
    	// t : time
        // y : type
        // d : depth
        // p : temperature
        // n : ndl
        // v : event value
    	
    	$(dive.samples).each(function(index, sample) {
    		if(sample.y != 1)
    			return true;
    		
			labels += '\"' + Math.floor(sample.t / 60) + '\'' + sample.t % 60 + '",';
			
			// depth...
			if(dive.distanceUnit == "Metric")			
				depth += roundOff(sample.d, 1) * -1 + ',';
			else
				depth += roundOff(M_FT(sample.d), 1) * -1 + ',';
					
			// temperature...			
			if(dive.temperatureUnit == "Celsius")
				temperature += roundOff(sample.p, 1) + ',';			
			else
				temperature += roundOff(C_F(sample.p), 1) + ',';				
		});    	
    }
    else {
    	if(dive.format == "sml") {
    		// tm : time
    		// dp : depth
    		// te : temperature	
    		// tp : tank pressure    
    		
    		$(dive.samples).each(function(index, sample) {
    			if(sample.dp == 0 && sample.te == null)
    				return true;
    				
    			labels += '\"' + Math.floor(sample.tm / 60) + '\'' + sample.tm % 60 + '",';
    			
    			// depth...
    			if(dive.distanceUnit == "Metric")			
    				depth += roundOff(sample.dp, 1) * -1 + ',';
    			else
    				depth += roundOff(M_FT(sample.dp), 1) * -1 + ',';
    					
    			// temperature...
    			if(sample.te != null) {
    				if(dive.temperatureUnit == "Celsius")
    					currentTemperature = roundOff(K_C(sample.te), 1);					
    				else
    					currentTemperature = roundOff(K_F(sample.te), 1);			
    			}
    			
    			if(currentTemperature > 0)
    				temperature += currentTemperature + ',';
    		});
    	}
    }	
	
	if(labels.length > 0)
		labels = eval("([" + labels.slice(0, -1) + "])");
	
	if(depth.length > 0)
		depth = eval("([" + depth.slice(0, -1) + "])");
	
	if(temperature.length > 0)
		temperature = eval("([" + temperature.slice(0, -1) + "])");
	
	profile.labels = labels;
	profile.depth = depth;
	profile.temperature = temperature;		
	
	return profile;
}


////////////////////////////////////////////////////////////////////////////////////////////
/// Generate Html
////////////////////////////////////////////////////////////////////////////////////////////

function setLogbookListView(dives, pageNumber) {
	var view = ''; 
	
	$(dives).each(function(index, dive) {
		view += getLogbookItemView(dive, true);
	});	
	
	$("#news_feed .post_section").empty();
	$("#news_feed .post_section").append(view);
	
	setLogbookPaging(pageNumber, g_logbookTotalPageCount);
	$(window).scrollTop(0);
}

function setLogbookBestListView(dives) {
	if(dives.length == 0) {
		$("#aside .popluar_dive").hide();
		return;
	}		
	
	var id = '';
	var view = '';
		
	view += '<h1>' + lang["logbook_popular"] + '</h1>';
	view += '<div class="list">';
	view += 	'<ul>';
	
	$(dives).each(function(index, dive) {
		id = getLogbookId(dive.userId, dive.logbookNumber);
		
		view += '<li>';
		view += 	'<a class="user_link_dir" href="' + dive.userId + '">';
		view += 		'<div class="user">';
		view += 			'<div class="avatar"><img src="' + dive.userImageLocation + '"/></div>';
		view += 			'<div class="info">';
		view += 				'<div class="nickname">' + dive.userName + '</div>';
		view += 				'<div class="date">' + dive.diveDate + '</div>';
		view += 			'</div>';
		view += 		'</div>';
		view += 	'</a>';
		view += 	'<a href="' + id + '" class="logbook_popular">';
		view += 		'<div class="latest">';
		view += 			'<div class="ellipsis"><div>';
		view += 				dive.location;
		view += 			'</div></div>';
		view += 		'</div>';
		view += 	'</a>';
		view += '</li>';
	});
		
	view += 	'</ul>';
	view += '</div>';
	
	$("#aside .popluar_dive").show();
	$("#aside .popluar_dive").empty();	
	$("#aside .popluar_dive").append(view);	
}

function setLogbookStatisticsOverviewView(data) {
	var view = '';
	
	var timeHour = 0; 
	var timeMinute = 0;
	
	if(data.totalDiveTime > 0) {
		timeHour = parseInt(data.totalDiveTime / 60);
		timeMinute = parseInt(data.totalDiveTime % 60);
	}	
		
	view += '<div class="statistics">';
	view +=	'<ul>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["total_log_count"] + '</h2>';
	view +=					'<div class="explain">' + getNumberToCurrency(data.totalLogCount) + '</div>';	
	view +=				'</div>';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["total_dive_time"] + '</h2>';
	view +=					'<div class="explain">' + timeHour + ':' + timeMinute + '</div>';
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="graph_view"></div>';
	view +=		'</li>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["log_status_per_month"] + '</h2>';
	view +=					'<div id="log_month" class="selector double"><select name="year"></select></div>';	
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="graph_view active">';
	view +=				'<div id="logbook_month_chart_wrap"></div>';
	view +=			'</div>';
	view +=		'</li>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["log_status_per_year"] + '</h2>';	
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="graph_view active">';
	view +=				'<div id="logbook_year_chart_wrap"></div>';
	view +=			'</div>';
	view +=		'</li>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["view_count_top"] + '</h2>';
	view +=				'</div>';
	view +=			'</div>';	
	view += 			'<div class="logbook_top view_top">';	
	view += 				'<div class="post_section"></div>';	
	view += 			'</div>';
	view +=			'<div class="graph_view"></div>';
	view +=		'</li>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["comment_count_top"] + '</h2>';
	view +=				'</div>';
	view +=			'</div>';		
	view += 			'<div class="logbook_top comment_top">';	
	view += 				'<div class="post_section"></div>';	
	view += 			'</div>';
	view +=			'<div class="graph_view"></div>';
	view +=		'</li>';
	view += 		'<li>';
	view +=			'<div class="clearfix">';
	view +=				'<div class="item">';
	view +=					'<h2>' + lang["like_count_top"] + '</h2>';
	view +=				'</div>';
	view +=			'</div>';		
	view += 			'<div class="logbook_top like_top">';	
	view += 				'<div class="post_section"></div>';	
	view += 			'</div>';
	view +=			'<div class="graph_view"></div>';
	view +=		'</li>';
	view += 	'</ul>';	
	view += '</div>';
	
	$("#statistics").append(view);
	
	// view top list...
	view = '';
	
	$(data.viewTopList).each(function(index, dive) {		
		view += getLogbookItemView(dive, false);
	});
	
	$(".view_top .post_section").append(view);
	
	// comment top list...
	view = '';
	
	$(data.commentTopList).each(function(index, dive) {		
		view += getLogbookItemView(dive, false);
	});
	
	$(".comment_top .post_section").append(view);
	
	// like top list...
	view = '';
	
	$(data.likeTopList).each(function(index, dive) {		
		view += getLogbookItemView(dive, false);
	});
	
	$(".like_top .post_section").append(view);
	
	requestLogbookStatisticsMinimumYear(g_userId);
	requestLogbookStatisticsPerYear(g_userId);
}

function setLogbookStatisticsPerYearView(raw) {
	var data = new google.visualization.DataTable();
	
	data.addColumn("string", "'Year");
	data.addColumn("number", "Count");
	data.addColumn({type: "number", role: "annotation"});
	data.addColumn({type: "string", role: "style"});
	
	$(raw).each(function(index, item) {
		data.addRow([item.target, parseInt(item.value), parseInt(item.value), "gold"]);
	});
	
	var view = new google.visualization.DataView(data);
    var options = {                       
    	bar: { groupWidth: "90%" },
    	legend: { position: 'none' }    
    };
    
    var chart = new google.visualization.ColumnChart(document.getElementById("logbook_year_chart_wrap"));
    chart.draw(view, options);          
}

function setLogbookStatisticsPerMonthView(raw) {	
	var data = new google.visualization.DataTable();
	
	data.addColumn("string", "'Month");
	data.addColumn("number", "Count");
	data.addColumn({type: "number", role: "annotation"});
	
	$(raw).each(function(index, item) {
		data.addRow([item.target, parseInt(item.value), parseInt(item.value)]);
	});	
	
	var view = new google.visualization.DataView(data);
    var options = {                       
    	bar: { groupWidth: "90%" },
    	legend: { position: 'none' }    
    };
    
    var chart = new google.visualization.ColumnChart(document.getElementById("logbook_month_chart_wrap"));
    chart.draw(view, options);                                              
}

function setLogbookDetailView(dive) {
	g_logbookSelectedItem = dive;
	
	var id = getLogbookId(dive.userId, dive.logbookNumber);	
	var noteWrap = '';
	var note = '';	
	var noteWrapObj = '';
	var noteObj = '';
	
	if(dive.inputType == "Imported" || dive.inputType == "Device") {
		noteWrap = "dive_memo";
		note = "note";
		noteWrapObj = ".dive_memo";
		noteObj = ".note";
	}
	else {
		noteWrap = "dive_memo_hand";
		note = "note_hand";
		noteWrapObj = ".dive_memo_hand";
		noteObj = ".note_hand";
	}
	
	var view = '';
	
	view += '<div class="layer_popup log_view" id="' + id + '">';
	view += 	'<div class="logbook">';
	view +=	 	'<div class="dive_info">';
	view +=			'<div class="dive_summary border_bottom">';
	view +=				'<div>';
	view +=					'<table>';
	view +=						'<tr>';
	view +=							'<td class="width70px bold">' + lang["dive_number"] + '</td><td class="width70px blue">' + dive.diveNumber + '</td>';
	view +=							'<td class="width50px bold">' + lang["dive_date"] + '</td><td class="blue">' + dive.diveDate + '</td>';
	view +=						'</tr>';
	view +=						'<tr><td class="bold">' + lang["location"] + '</td><td colspan="3" class="blue">' + dive.location + '</td></tr>';
	view +=						'<tr><td class="bold">' + lang["point"] + '</td><td colspan="3" class="blue">' + dive.point + '</td></tr>';
	view +=					'</table>';
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="bathymeter">';
	view +=				'<div class="bathymeter_top">';
	view +=					'<div>';
	view +=						'<span class="bold width24_pc_mobile50 title">' + lang["si"] + '</span><span class="width12_pc_mobile50">' + getSurfaceInterval(dive) + '</span>';
	view +=						'<span class="bold width24_pc_mobile30 title">' + lang["dive_time"] + '</span><span class="width12_pc_mobile20">' + dive.diveTime + ' min</span>';
	view +=						'<span class="bold width18_pc_mobile35 title">' + lang["safety_stop_time"] + '</span><span class="width10_pc">' + dive.safetyStopTime + ' min</span>';
	view +=						'<span class="bold width24_pc_mobile30 title">' + lang["max_depth"] + '</span><span class="width12_pc_mobile20">' + dive.maxDepth + getDistanceUnit(dive.distanceUnit) + '</span>';
	view +=						'<span class="bold width24_pc_mobile35 title">' + lang["average_depth"] + '</span><span class="width12_pc">' + dive.averageDepth + getDistanceUnit(dive.distanceUnit) + '</span>';
	view +=					'</div>';
	view +=				'</div>';
	view +=				'<div class="bathymeter_bottom">';
	view +=					'<div class="bathymeter_left">';
	view +=						'<div>';
	view +=							'<div class="desc">';
	view +=								'<div class="desc_top">';
	view +=									'<span class="bold title_blue width35">' + lang["time_text"] + '</span><span id="graph_time_value">--</span>';
	view +=									'<span class="bold title_blue width25">' + lang["depth_text"] + '</span><span id="graph_distance_value">--' + getDistanceUnit(dive.distanceUnit) + '</span>';
	view +=									'<span class="bold title_blue width35">' + lang["water_temperature_text"] + '</span><span id="graph_temperature_value">--' + getTemperatureUnit(dive.temperatureUnit) + '</span>';
	view +=									'<span class="bold title_blue width25">' + lang["tank_text"] + '</span><span id="graph_pressure_value">--' + getPressureUnit(dive.pressureUnit) + '</span>';
	view +=								'</div>';
	view +=							'</div>';
	view +=							'<div id="graph_view_canvas_wrap">';
	view +=								'<canvas id="graph_view_canvas"></canvas>';
	view +=							'</div>';
	view +=						'</div>';
	view +=					'</div>';
	view +=					'<div class="bathymeter_left_hand">';
	view +=						'<div>';
	view +=							'<div class="Surface_Interval"><span class="bold">' + lang["si"] + '</span><span>' + getSurfaceInterval(dive) + '</span></div>';
	view +=							'<div class="Dive_Time"><span class="bold block">' + lang["dive_time"] + '</span>' + dive.diveTime + ' min</div>';
	view +=							'<div class="Safety_Stop"><span class="bold block">' + lang["safety_stop_time"] + '</span>' + dive.safetyStopTime + ' min</div>';
	view +=							'<div class="Max_Depth"><span class="bold block">' + lang["max_depth"] + '</span>' + dive.maxDepth + getDistanceUnit(dive.distanceUnit) + '</div>';
	view +=							'<div class="Average_Depth"><span class="bold block">' + lang["average_depth"] + '</span>' + dive.averageDepth + getDistanceUnit(dive.distanceUnit) + '</div>';
	view +=						'</div>';
	view +=					'</div>';
	view +=					'<div class="bathymeter_right">';
	view +=						'<div class="overflow">';
	view +=							'<div class="time">';
	view +=								'<div>';
	view +=									'<table>';
	view +=										'<tr><td rowspan="3" class="background_time" width="60px"></td><td colspan="2" class="bold title">' + lang["time"] + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["time_in"] + '</td><td>' + getTime(dive.timeInHour, dive.timeInMinute) + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["time_out"] + '</td><td>' + getTime(dive.timeOutHour, dive.timeOutMinute) + '</td></tr>';									
	view +=									'</table>';
	view +=								'</div>';
	view +=							'</div>';
	view +=							'<div class="pressure">';
	view +=								'<div>';
	view +=									'<table>';
	view +=										'<tr><td rowspan="3" class="background_pressure" width="60px"></td><td colspan="2" class="bold title">' + lang["tank_pressure"] + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["start_tank_pressure"] + '</td><td>' + dive.startTankPressure + getPressureUnit(dive.pressureUnit) + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["end_tank_pressure"] + '</td><td>' + dive.endTankPressure + getPressureUnit(dive.pressureUnit) + '</td></tr>';
	view +=									'</table>';
	view +=								'</div>';
	view +=							'</div>';
	view +=						'</div>';						
	view +=						'<div class="overflow">';
	view +=							'<div class="temperature">';
	view +=								'<div>';
	view +=									'<table>';
	view +=										'<tr><td rowspan="3" class="background_temp" width="60px"><td colspan="2" class="bold title">' + lang["temperature"] + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["max_temperature"] + '</td><td>' + dive.maxTemperature + getTemperatureUnit(dive.temperatureUnit) + '</td></tr>';
	view +=										'<tr><td class="bold">' + lang["min_temperature"] + '</td><td>' + dive.minTemperature + getTemperatureUnit(dive.temperatureUnit) + '</td></tr>';
	view +=									'</table>';
	view +=								'</div>';
	view +=							'</div>';	
	view +=						'</div>';						
	view +=					'</div>';
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="planning">';
	view +=				'<div>';
	view +=					'<span class="bold title width20">' + lang["planning_tool"] + '</span><span class="width20">' + getDivePlanningTool(dive.divePlanningTool) + '</span>';
	view +=					'<span class="bold title width16">' + lang["weight" ] + '</span><span class="width16">' + dive.weight + getWeightUnit(dive.weightUnit) + '</span>';
	view +=					'<span class="bold title">' + lang["enriched_air_blend" ] + '</span><span>' + getEnrichedAirBlend(dive) + '</span>';
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="visibility">';
	view +=				'<div>';
	view +=				 	'<div class="border_bottom_pc overflow">';
	view +=						'<div class="right width75">';
	view +=							'<span class="bold title width40">' + lang["exposure_protection"] + '</span><span class="width60">' + getExposureProtection(dive) + '</span>';
	view +=						'</div>';
	view +=						'<div class="right width25">';
	view +=							'<span class="bold title width70">' + lang["visibility"] + '</span><span class="width30">' + dive.visibility + getDistanceUnit(dive.distanceUnit) + '</span>';
	view +=						'</div>';
	view +=					'</div>';
	view +=				 	'<div class="visibility_bottom overflow">';
	view +=					 	'<span class="bold title width13">' + lang["dive_type"] + '</span><span class="width12">' + getDiveType(dive) + '</span>';
	view +=						'<span class="bold title width13">' + lang["water_type"] + '</span><span class="width12">' + getWaterType(dive) + '</span>';
	view +=					 	'<span class="bold title width13">' + lang["wave_type"] + '</span><span class="width12">' + getStrengthType(dive.waveType) + '</span>';
	view +=						'<span class="bold title width13">' + lang["current_type"] + '</span><span class="width12">' + getStrengthType(dive.currentType) + '</span>';
	view +=				 	'</div>';
	view +=				 '</div>';
	view +=			'</div>';
	view +=			'<div class="' + noteWrap + '">';
	view +=				'<div>';
	view +=					'<div class="border_bottom line_height35px">';
	view +=						'<div class="bold width67px">' + lang["activity"] + '</div>';
	view +=						'<div>' + dive.activity + '</div>';
	view +=					'</div>';
	view +=					'<div class="textarea">';
	view +=						'<div class="bold width67px">' + lang["note"] + '</div>';
	view +=						'<div class="' + note + '">' + getText(dive.note) + '</div>';
	view +=					'</div>';
	view +=					'<div class="border_top sticker">';
	view +=						'<div class="bold width67px">' + lang["sticker"] + '</div>';
	view +=						'<div class="sticker_image">';
	view +=							getStickers(dive.stickers);
	view +=						'</div>';
	view +=					'</div>';
	view +=				'</div>';
	view +=			'</div>';
	view +=			'<div class="member_pos">';			
	view +=				'<div class="user">';	
	view +=					getSignature(dive);
	view +=				'</div>';
	view +=				'<div class="ad_frame">';	
	view +=					'<div>';
	view +=						'<a href="index.jsp"><img src="./main/images/default_ad_logbook.jpg" /></a>';
	view +=					'</div>';
	view +=				'</div>';
	view +=			'</div>';	
	view += 		'</div>';	
	view += 		'<div class="dive_comments">';
	view += 			'<div class="photo_list">';
	view +=				getLogbookThumbnailImageView(dive, "", true);
	view += 				'<div class="control">';
	view += 					'<div class="response">';
	view += 						'<a class="logbook_like_btn" href="' + id + '">' + lang["like"] + ' <b>' + dive.likeCount + '</b></a>';
	view += 						'<a class="logbook_comment_btn" href="' + id + '">' + lang["comment"] + ' <b>' + dive.commentCount + '</b></a>';
	view += 					'</div>';
	view +=					getLogbookItemMenu(dive, true);	
	view += 				'</div>';
	view += 			'</div>';
	view += 			'<div class="comments_write">';
	view += 			'</div>';
	view += 			'<div class="comments">';
	view += 				'<div id="iscroll">';
	view += 					'<div id="scroller">';
	view += 						'<div class="post_list">';								
	view += 						'</div>';
	view += 					'</div>';
	view += 				'</div>';
	view += 				'<div class="post_pager">';
	view += 				'</div>';
	view += 			'</div>';
	view += 		'</div>';
	view += 	'</div>';
	view += '</div>';
	
	g_logbookGalleryView = getLogbookGalleryView(dive.fileList);
	
	showPopup(view);
	autolink($(noteObj));
		
	if(dive.inputType == "Imported" || dive.inputType == "Device") {
		$(".bathymeter_top").show();
		$(".bathymeter_left").show();
		$(".bathymeter_left_hand").hide();
		
		setLogbookProfileView(dive, $("#graph_view_canvas"));
	}	
	else {
		$(".bathymeter_top").hide();
		$(".bathymeter_left").hide();
		$(".bathymeter_left_hand").show();
	}
	
	if($(window).height() < 768) {
		var popup = $(".log_view");
		var height = $(window).height() - 40;
		var gap = 768 - height;
		
		noteObj = ".log_view " + noteObj;
		noteWrapObj = ".log_view " + noteWrapObj;
		
		popup.css("height", height);
		
		$(noteWrapObj).css("height", $(noteWrapObj).height() - gap);
		$(noteObj).css("height", $(noteObj).height() - gap);
		$(".dive_comments").css("height", $(".dive_comments").height() - gap + 30);
		$(".dive_comments .comments").css("height", $(".dive_comments .comments").height() - gap);
		$(".dive_comments #iscroll").css("height", $(".dive_comments #iscroll").height() - gap);
	}	
	
	var writable = false;	
	if(dive.userId == g_myId || dive.buddyStatus == "Accepted" || dive.buddyStatus == "Favorite")		
		writable = true;
	
	if(g_logbookIsPostedAd == true)
		writable = true;	
	else
		requestAdPickKeyword("Keyword_Logbook", dive.location + " " + dive.point + " " + dive.activity);
	
	initLogbookComment(dive.userId, dive.logbookNumber, dive.commentCount, writable);	
}

function setLogbookEditView(type, dive) {	
	var id = getLogbookId(dive.userId, dive.logbookNumber);
	var form = '';
	var submitText = '';
	
	if(type == 'new' || type == 'copy') {
		form = '<form name="logbook_write_form">';
		submitText = lang["write"];
		g_logbookPrevAd = false;
	}
	else {
		if(dive.signatureStatus == "Put")
			type = 'modify_restrict';
		
		if(type == 'modify')
			form = '<form name="logbook_modify_form" action="' + id + '">';
		else
			form = '<form name="logbook_modify_restricted_form" action="' + id + '">';
		
		submitText = lang["modify"];
		g_logbookPrevAd = dive.isAd;
	}	
	
	var distanceUnit = getDistanceUnit(dive.distanceUnit);
	var weightUnit = getWeightUnit(dive.weightUnit);
	var temperatureUnit = getTemperatureUnit(dive.temperatureUnit);
	var pressureUnit = getPressureUnit(dive.pressureUnit);
	
	var view = '';
	
	view += '<h1>' + lang["logbook"] + '</h1>';
	view += form;
	view += 	'<div class="dive_info">';
	view += 		'<div class="dive_setting">';
	view += 			'<button type="button">' + lang["setting_unit"] + '</button>';
	view += 			'<div class="control_box">';
	view += 				'<table width="100%">';
	view += 					'<colgroup>';
	view += 						'<col width="70" />';
	view += 						'<col width="55" />';
	view += 						'<col width="90" />';
	view += 						'<col width="55" />';
	view += 					'</colgroup>';
	view += 					'<tbody>';
	view += 						'<tr>';
	view += 							'<th><b>' + lang["distance_unit"] + '</b></th>';
	view += 							'<td>';
	view += 								'<div class="selector">';
	view += 									'<select name="distanceUnit">';
	view += 										'<option value="Metric" ' +  getSelected(dive.distanceUnit, "Metric") + '>m</option>';
	view += 										'<option value="Feet" ' +  getSelected(dive.distanceUnit, "Feet") + '>ft</option>';	
	view += 									'</select>';
	view += 								'</div>';										
	view += 							'</td>';
	view += 							'<th><b>' + lang["temperature_unit"] + '</b></th>';
	view += 							'<td>';
	view += 								'<div class="selector">';
	view += 									'<select name="temperatureUnit">';
	view += 										'<option value="Celsius" ' +  getSelected(dive.temperatureUnit, "Celsius") + '>°C</option>';
	view += 										'<option value="Fahrenheit" ' +  getSelected(dive.temperatureUnit, "Fahrenheit") + '>°F</option>';
	view += 									'</select>';
	view += 								'</div>';										
	view += 							'</td>';
	view += 						'</tr>';
	view += 						'<tr>';
	view += 							'<th><b>' + lang["weight_unit"] + '</b></th>';
	view += 							'<td>';
	view += 								'<div class="selector">';
	view += 									'<select name="weightUnit">';
	view += 										'<option value="Kilogram" ' +  getSelected(dive.weightUnit, "Kilogram") + '>kg</option>';
	view += 										'<option value="Pound" ' +  getSelected(dive.weightUnit, "Pound") + '>lbs</option>';
	view += 									'</select>';
	view += 								'</div>';										
	view += 							'</td>';
	view += 							'<th><b>' + lang["pressure_unit"] + '</b></th>';
	view += 							'<td>';
	view += 								'<div class="selector">';
	view += 									'<select name="pressureUnit">';
	view += 										'<option value="Bar" ' +  getSelected(dive.pressureUnit, "Bar") + '>bar</option>';
	view +=										'<option value="Psi" ' +  getSelected(dive.pressureUnit, "Psi") + '>psi</option>';
	view += 									'</select>';
	view += 								'</div>';										
	view += 							'</td>';
	view += 						'</tr>';
	view += 					'</tbody>';									
	view += 				'</table>';
	view += 			'</div>';
	view += 		'</div>';
	view += 		'<div class="dive_summary">';
	view += 			'<table>';
	view += 				'<colgroup>';
	view += 					'<col width="60" />';
	view += 					'<col width="100" />';
	view += 					'<col width="40" />';
	view += 					'<col width="" />';
	view += 				'</colgroup>';
	view += 				'<tbody>';
	view += 					'<tr>';
	view += 						'<th scope="col">' + lang["dive_number"] + '</th>';
	view += 						'<td scope="col"><input type="text" name="diveNumber" class="short" value="' + dive.diveNumber + '"/></td>';
	view += 						'<th scope="col">' + lang["dive_date"] + '</th>';
	view += 						'<td scope="col">';
	view += 							'<div class="date_pick">';
	view += 								'<input type="text" name="diveDate" readonly="readonly" id="datepicker_logbook" class="short" />';	
	view += 							'</div>';
	view += 						'</td>';
	view += 					'</tr>';
	view += 					'<tr>';
	view += 						'<th scope="col">' + lang["location"] + '</th>';
	view += 						'<td scope="col" colspan="3"><input type="text" name="location" class="full" /></td>';
	view += 					'</tr>';
	view += 					'<tr>';
	view += 						'<th scope="col">' + lang["point"] + '</th>';
	view += 						'<td scope="col" colspan="3"><input type="text" name="point" class="full" /></td>';
	view += 					'</tr>';
	view += 				'</tbody>';
	view += 			'</table>';
	view += 		'</div>';
	view +=		'<div id="logbook_import_area" class="upload_description">';
	view +=			'<div class="upload_text"><span class="bold">' + lang["logbook_import_info"] + '</span><span class="margin_left"><a class="logbook_help_import_btn" href="#">[' + lang["help"] + ']</a></span></div>';	
	view +=			'<div class="filebox">';
	view +=				'<label for="log_files">' + lang["choose_file"] + '</label>';
	view +=				'<input type="file" id="log_files" name="file" onchange="readLogFiles(\'' + type + '\', 0, null);">';
	view +=			'</div>';	
	view +=		'</div>';
	view += 		'<div class="dive_detail">';
	view += 			'<div class="dive_note">';
	view += 				'<div class="bathymeter">';
	view +=					'<div id="auto_edit_graph">';
	view +=						'<div id="graph_edit_canvas_wrap">';
	view +=							'<canvas id="graph_edit_canvas"></canvas>';
	view +=						'</div>';
	view +=						'<div class="desc data_table">';
	view +=							'<table summary="상세 정보">';
	view +=								'<colgroup>';
	view +=									'<col width="120px"/>';
	view +=									'<col width="40px"/>';
	view +=								'</colgroup>';
	view +=								'<tbody>';							
	view +=									'<tr>';
	view +=										'<th scope="col"><b>' + lang["si"] + '</b></th>';
	view +=										'<td scope="col"><input type="text" readonly="readonly" value="' + getSurfaceInterval(dive) + '"/></td>';
	view +=									'</tr>';
	view +=									'<tr>';
	view +=										'<th scope="col"><b>' + lang["max_depth"] + '</b></th>';
	view +=										'<td scope="col"><input type="text" readonly="readonly" value="' + dive.maxDepth + getDistanceUnit(dive.distanceUnit) + '"/></td>';
	view +=									'</tr>';
	view +=									'<tr>';
	view +=										'<th scope="col"><b>' + lang["average_depth"] + '</b></th>';
	view +=										'<td scope="col"><input type="text" readonly="readonly" value="' + dive.averageDepth + getDistanceUnit(dive.distanceUnit) + '" /></td>';
	view +=									'</tr>';
	view +=									'<tr>';
	view +=										'<th scope="col"><b>' + lang["dive_time"] + '</b></th>';
	view +=										'<td scope="col"><input type="text" readonly="readonly" value="' + dive.diveTime + ' min" /></td>';
	view +=									'</tr>';
	view +=									'<tr>';
	view +=										'<th scope="col"><b>' + lang["safety_stop_time"] + '</b></th>';
	view +=										'<td scope="col"><input type="text" class="short" name="safetyStopTime_auto" value="' + dive.safetyStopTime + '"/> min</td>';
	view +=									'</tr>';
	view += 									'<input type="hidden" name="format"/>';
	view += 									'<input type="hidden" name="deviceName"/>';
	view += 									'<input type="hidden" name="deviceVersion"/>';
	view += 									'<input type="hidden" name="serialNumber"/>';
	view += 									'<input type="hidden" name="altitude"/>';
	view += 									'<input type="hidden" name="sampleInterval"/>';
	view += 									'<input type="hidden" name="samples"/>';
	view += 									'<input type="hidden" name="data"/>';
	view +=								'</tbody>';
	view +=							'</table>';
	view +=						'</div>';
	view +=					'</div>';
	view +=					'<div id="manual_edit_graph">';
	view += 						'<div class="info_si">';
	view += 							'<b>' + lang["si"] + '</b> <input type="text" class="short" name="surfaceIntervalHour" /> : <input type="text" class="short" name="surfaceIntervalMinute" />';	
	view += 						'</div>';	
	view += 						'<div class="info_depth">';
	view += 							'<b>' + lang["max_depth"] + '</b>';
	view += 							'<input type="text" class="double" name="maxDepth" /> ' + '<span class="distanceUnit">' + distanceUnit + '</span>';
	view += 						'</div>';
	view += 						'<div class="info_safety">';
	view += 							'<b>' + lang["safety_stop_time"] + '</b>';
	view += 							'<input type="text" class="double" name="safetyStopTime" /> min';
	view += 						'</div>';
	view += 						'<div class="info_averdge">';
	view += 							'<b>' + lang["average_depth"] + '</b>';
	view += 							'<input type="text" class="double" name="averageDepth" /> ' + '<span class="distanceUnit">' + distanceUnit + '</span>';
	view += 						'</div>';
	view += 						'<div class="info_time">';
	view += 							'<b>' + lang["dive_time"] + '</b>';
	view += 							'<input type="text" class="double dive_time" name="diveTime" /> min';
	view += 						'</div>';
	view +=					'</div>';
	view += 				'</div>';
	view += 				'<div class="time data_table">';
	view += 					'<table>';
	view += 						'<colgroup>';
	view += 							'<col width="" />';
	view += 							'<col width="60" />';
	view += 						'</colgroup>';
	view += 						'<tbody>';
	view += 							'<tr>';
	view += 								'<th colspan="2" scope="row"><b>' + lang["time"] + '</b></th>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th scope="col">' + lang["time_in"] + '</th>';
	view += 								'<td scope="col">';
	view += 									'<input type="text" class="short time_in_hour" name="timeInHour"/> : <input type="text" class="short time_in_minute" name="timeInMinute"/>';
	view += 								'</td>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th scope="col">' + lang["time_out"] + '</th>';
	view += 								'<td scope="col">';
	view += 									'<input type="text" class="short" name="timeOutHour"/> : <input type="text" class="short" name="timeOutMinute"/>';
	view += 								'</td>';
	view += 							'</tr>';
	view += 						'</tbody>';
	view += 					'</table>';
	view += 				'</div>';
	view += 				'<div class="pressure data_table">';
	view += 					'<table>';
	view += 						'<colgroup>';
	view += 							'<col width="" />';
	view += 							'<col width="60" />';
	view += 						'</colgroup>';
	view += 						'<tbody>';
	view += 							'<tr>';
	view += 								'<th colspan="2" scope="row"><b>' + lang["tank_pressure"] + '</b></th>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th scope="col">' + lang["start_tank_pressure"] + '</th>';
	view += 								'<td scope="col"><input type="text" class="double" name="startTankPressure"/> ' + '<span class="pressureUnit">' + pressureUnit + '</span>' + '</td>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th scope="col">' + lang["end_tank_pressure"] + '</th>';
	view += 								'<td scope="col"><input type="text" class="double" name="endTankPressure"/> ' + '<span class="pressureUnit">' + pressureUnit + '</span>' + '</td>';
	view += 							'</tr>';
	view += 						'</tbody>';
	view += 					'</table>';
	view += 				'</div>';
	view += 			'</div>';
	view += 			'<div class="dive_note">';
	view += 				'<div class="planning data_table">';
	view += 					'<div class="outline">';
	view += 						'<table>';
	view += 							'<colgroup>';
	view += 								'<col width="100" />';
	view += 								'<col width="" />';
	view += 								'<col width="70" />';
	view += 								'<col width="90" />';
	view += 								'<col width="50" />';
	view += 								'<col width="70" />';
	view += 							'</colgroup>';
	view += 							'<tbody>';
	view += 								'<tr>';
	view += 									'<th><b>' + lang["planning_tool"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="divePlanningTool">';
	view += 												'<option value="Computer" ' + getSelected(dive.divePlanningTool, "Computer") + '>' + lang["planning_tool_computer"] + '</option>';
	view += 												'<option value="Table" ' + getSelected(dive.divePlanningTool, "Table") + '>' + lang["planning_tool_table"] + '</option>';
	view += 												'<option value="Other" ' + getSelected(dive.divePlanningTool, "Other") + '>' + lang["planning_tool_other"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 									'</td>';
	view += 									'<th><b>' + lang["weight"] + '</b></th>';
	view += 									'<td>';
	view += 										'<input type="text" class="double" name="weight" value="' + dive.weight + '" /> ' + '<span class="weightUnit">' + weightUnit + '</span>';
	view += 									'</td>';
	view += 									'<th><b>' + lang["enriched_air_blend"] + '</b></th>';
	view += 									'<td>';
	view += 										'<input type="text" class="double" name="enrichedAirBlend" /> %';
	view += 									'</td>';
	view += 								'</tr>';
	view += 							'</tbody>';
	view += 						'</table>';
	view += 					'</div>';
	view += 					'<div class="outline">';
	view += 						'<table>';
	view += 							'<colgroup>';
	view += 								'<col width="130" />';
	view += 								'<col width="" />';
	view += 							'</colgroup>';							
	view += 							'<tbody>';
	view += 								'<tr>';
	view += 									'<th><b>' + lang["exposure_protection"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="suitType">';
	view += 												'<option value="Skin" ' + getSelected(dive.suitType, "Skin") + '>' + lang["suit_type_skin"] + '</option>';
	view += 												'<option value="Wet" ' + getSelected(dive.suitType, "Wet") + '>' + lang["suit_type_wet"] + '</option>';
	view += 												'<option value="Dry" ' + getSelected(dive.suitType, "Dry") + '>' + lang["suit_type_dry"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 										'<div class="option">';
	view += 											'<label><input type="checkbox" name="hood" ' + getCheckedExposureProtection(dive, "hood") + '/>' + lang["hood"] + '</label>';
	view += 											'<label><input type="checkbox" name="gloves" ' + getCheckedExposureProtection(dive, "gloves") + '/>' + lang["gloves"] + '</label>';
	view += 											'<label><input type="checkbox" name="boots" ' + getCheckedExposureProtection(dive, "boots") + '/>' + lang["boots"] + '</label>';
	view += 										'</div>';
	view += 									'</td>';
	view += 								'</tr>';
	view += 							'</tbody>';
	view += 						'</table>';
	view += 					'</div>';
	view += 				'</div>';
	view += 			'</div>';
	view += 			'<div class="dive_note">';
	view += 				'<div class="visibility data_table">';
	view += 					'<div class="outline">';
	view += 						'<table>';
	view += 							'<colgroup>';
	view += 								'<col width="75" />';
	view += 								'<col width="" />';
	view += 							'</colgroup>';
	view += 							'<tbody>';
	view += 								'<tr>';
	view += 									'<th><b>' + lang["visibility"] + '</b></th>';
	view += 									'<td>';
	view += 										'<input type="text" class="double" name="visibility" /> ' + '<span class="distanceUnit">' + distanceUnit + '</span>';
	view += 									'</td>';
	view += 								'</tr>';
	view += 							'</tbody>';
	view += 					'</table>';
	view += 					'</div>';
	view += 					'<div class="outline">';
	view += 						'<table>';
	view += 							'<colgroup>';
	view += 								'<col width="75" />';
	view += 								'<col width="85" />';
	view += 								'<col width="85" />';
	view += 								'<col width="" />';
	view += 							'</colgroup>';
	view += 							'<tbody>';
	view += 								'<tr>';
	view += 									'<th><b>' + lang["dive_type"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="diveType">';
	view += 												'<option value="Shore" ' + getSelected(dive.diveType, "Shore") + '>' + lang["dive_type_shore"] + '</option>';
	view += 												'<option value="Boat" ' + getSelected(dive.diveType, "Boat") + '>' + lang["dive_type_boat"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 									'</td>';
	view += 									'<th><b>' + lang["water_type"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="waterType">';
	view += 												'<option value="Salt" ' + getSelected(dive.waterType, "Salt") + '>' + lang["water_type_salt"] + '</option>';
	view += 												'<option value="Fresh" ' + getSelected(dive.waterType, "Fresh") + '>' + lang["water_type_fresh"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 									'</td>';
	view += 								'</tr>';
	view += 							'</tbody>';
	view += 						'</table>';
	view += 					'</div>';
	view += 					'<div class="outline">';
	view += 						'<table>';
	view += 							'<colgroup>';
	view += 								'<col width="75" />';
	view += 								'<col width="85" />';
	view += 								'<col width="85" />';
	view += 								'<col width="" />';
	view += 							'</colgroup>';
	view += 							'<tbody>';
	view += 								'<tr>';
	view += 									'<th><b>' + lang["wave_type"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="waveType">';
	view += 												'<option value="None" selected>' + lang["strength_none"] + '</option>';
	view += 												'<option value="Normal">' + lang["strength_normal"] + '</option>';
	view += 												'<option value="Strong">' + lang["strength_strong"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 									'</td>';
	view += 									'<th><b>' + lang["current_type"] + '</b></th>';
	view += 									'<td>';
	view += 										'<div class="selector double">';
	view += 											'<select name="currentType">';
	view += 												'<option value="None" selected>' + lang["strength_none"] + '</option>';
	view += 												'<option value="Normal">' + lang["strength_normal"] + '</option>';
	view += 												'<option value="Strong">' + lang["strength_strong"] + '</option>';
	view += 											'</select>';
	view += 										'</div>';
	view += 									'</td>';
	view += 								'</tr>';
	view += 							'</tbody>';
	view += 						'</table>';
	view += 					'</div>';
	view += 				'</div>';
	view += 				'<div class="temperature">';
	view += 					'<table>';
	view += 						'<colgroup>';
	view += 							'<col width="" />';
	view += 							'<col width="60" />';
	view += 						'</colgroup>';
	view += 						'<tbody>';
	view += 							'<tr>';
	view += 								'<th><b>' + lang["temperature"] + '</b></th>';
	view += 								'<td></td>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th>' + lang["max_temperature"] + '</th>';
	view += 								'<td><input type="text" class="double" name="maxTemperature" /> ' + '<span class="temperatureUnit">' + temperatureUnit + '</span>' + '</td>';
	view += 							'</tr>';
	view += 							'<tr>';
	view += 								'<th>' + lang["min_temperature"] + '</th>';
	view += 								'<td><input type="text" class="double" name="minTemperature" /> ' + '<span class="temperatureUnit">' + temperatureUnit + '</span>' + '</td>';
	view += 							'</tr>';
	view += 						'</tbody>';
	view += 					'</table>';
	view += 				'</div>';
	view += 			'</div>';
	view += 			'<div class="dive_memo">';
	view += 				'<table>';
	view += 					'<colgroup>';
	view += 						'<col width="70" />';
	view += 						'<col width="" />';
	view += 					'</colgroup>';
	view += 					'<tbody>';
	view += 						'<tr>';
	view += 							'<th>' + lang["activity"] + '</th>';
	view += 							'<td><input type="text" name="activity" /></td>';
	view += 						'</tr>';
	view += 						'<tr class="note">';
	view += 							'<th>' + lang["note"] + '</th>';
	view += 							'<td>';
	view += 								'<textarea cols="30" rows="10" name="note"></textarea>';
	view += 							'</td>';
	view += 						'</tr>';
	view += 						'<tr class="sticker">';
	view +=							'<input type="hidden" name="stickers"/>';
	view += 							'<th>';
	view +=								lang["sticker"];
	view += 								'<button type="button" name="logbook_sticker_view_btn">' + lang["append"] + '</button>';
	view += 							'</th>';
	view += 							'<td>';
	view += 								'<div class="sticker_list">';	 
	view +=									'<ul></ul>';	
	view += 								'</div>';
	view += 							'</td>';
	view += 						'</tr>';
	view += 					'</tbody>';
	view += 				'</table>';
	view += 			'</div>';
	view += 		'</div>';
	view += 		'<div class="post_write">';
	view += 			'<div class="update">';
	view += 				'<div class="function">';
	view += 					'<div class="aside">';
	view += 						'<div class="selector">';
	view += 							'<select name="viewAuthority">';
	view += 								'<option value="Public" ' +  getSelected(dive.viewAuthority, "Public") + '>' + lang["authority_public"] + '</option>';
	view += 								'<option value="Buddy" ' +  getSelected(dive.viewAuthority, "Buddy") + '>' + lang["authority_buddy"] + '</option>';
	view += 								'<option value="Private" ' +  getSelected(dive.viewAuthority, "Private") + '>' + lang["authority_private"] + '</option>';
	view += 							'</select>';
	view += 						'</div>';
	view += 					'</div>';
	view += 					'<div class="bside">';
	view += 						'<div class="verification">';	
	view += 							'<label for="verification">' + lang["signature"] + '&nbsp;</label>';
	view += 							'<div class="buddy_pick">';
	view += 								'<input type="text" id="verification" name="signeeName" readonly="readonly" />';	
	view += 								'<input type="image" name="findSigneeBtn" src="./main/images/btn_buddy_pick_not_requested.jpg" />';
	view += 								'<input type="hidden" name="signeeId"/>';
	view += 							'</div>';
	view += 						'</div>';
	view += 					'</div>';
	view += 				'</div>';
	view += 				'<input type="hidden" name="deviceType" value="Web"/>';			
	view +=				'<input type="hidden" name="inputType" value="' + dive.inputType + '"/>';
	view += 				'<div class="function">';
	view +=					'<input type="file" name="files" id="upload_logbook"/>';	
	view += 					'<div class="ad_export"><label><input type="checkbox" name="isAd" ' + getCheckbox(dive.isAd) + '/>' + lang["ad_export"] + '</label></div>';
	//view +=					'<input type="image" src="./main/images/btn_video_link.png" name="logbook_video_link_btn" />';
	view += 				'</div>';
	view +=				'<div class="upload_status logbook_video_link">';
	view +=					'<ul></ul>';
	view +=				'</div>';
	view +=				getLogbookUploadedFilesView(type, dive, id);
	view += 				'<div class="function">';
	view += 					'<div class="bside">';
	view += 						'<div class="submit">';
	view += 							'<input type="submit" value="' + submitText + '" />&nbsp;';	
	view += 							'<input type="reset" class="logbook_write_cancel_btn" value="' + lang["cancel"] + '" />';
	view += 						'</div>';
	view += 					'</div>';
	view += 				'</div>';								
	view += 			'</div>';
	view += 		'</div>';						
	view += 	'</div>';
	view += '</form>';
	
	var target = $("#log_book");
	
	target.empty();
	target.append(view);
	
	if(g_myType != 'Sponsor')
		$(".ad_export").hide();
	
	if(type != 'modify_restrict') {
		$("#datepicker_logbook").trigger("click");
		$("input[name=diveNumber]").trigger("click");
		$("#datepicker_logbook").datepicker("setDate" , dive.diveDate);
	}	
	
	$(".logbook_video_link").hide();
	$("#upload_logbook").trigger("click");	
	
	var findSigneeBtn = target.find("input[name=findSigneeBtn]");
	
	findSigneeBtn.attr("disabled", false);	
	findSigneeBtn.attr("src", "./main/images/btn_buddy_pick_not_requested.jpg");
	
	if(dive.inputType == "Imported" || dive.inputType == "Device") {
		$("#auto_edit_graph").show();
		$("#manual_edit_graph").hide();
		
		setLogbookProfileEditView(dive, $("#graph_edit_canvas"));
		
		target.find("input[name=maxTemperature]").attr('readonly', 'readonly');
		target.find("input[name=minTemperature]").attr('readonly', 'readonly');
		
		if(dive.usedTransmitter) {
			target.find("input[name=startTankPressure]").attr('readonly', 'readonly');
			target.find("input[name=endTankPressure]").attr('readonly', 'readonly');	
		}
		
		if(dive.inputType == "Device")
			$("#logbook_import_area").hide();
		
		/*
		if(type == 'copy') {
			$("#logbook_import_area").hide();
			
			target.find("input[name=format]").val(dive.format);
			target.find("input[name=deviceName]").val(dive.deviceName);
			target.find("input[name=deviceVersion]").val(dive.deviceVersion);
			target.find("input[name=serialNumber]").val(dive.serialNumber);
			target.find("input[name=altitude]").val(dive.altitude);
			target.find("input[name=sampleInterval]").val(dive.sampleInterval);
			target.find("input[name=sampleInterval]").val(dive.sampleInterval);
			target.find("select[name=divePlanningTool]").val("Computer");				
			
			if(dive.usedTransmitter) {
				$(dive.samples).each(function(index, sample) {
					delete sample.tp;
				});
			}
			
			target.find("input[name=samples]").val(JSON.stringify(dive.samples));
		}
		*/
	}
	else {
		$("#auto_edit_graph").hide();
		$("#manual_edit_graph").show();
	}
	
	$(window).scrollTop(0);
	
	if(type == 'new')
		return;
		
	var diveNumber = target.find("input[name=diveNumber]");
	var diveDate = target.find("input[name=diveDate]");
	var location = target.find("input[name=location]");
	var point = target.find("input[name=point]");		
	var surfaceIntervalHour = target.find("input[name=surfaceIntervalHour]");
	var surfaceIntervalMinute = target.find("input[name=surfaceIntervalMinute]");
	var maxDepth = target.find("input[name=maxDepth]");
	var averageDepth = target.find("input[name=averageDepth]");
	var diveTime = target.find("input[name=diveTime]");
	var safetyStopTime = target.find("input[name=safetyStopTime]");
	var safetyStopTime_auto = target.find("input[name=safetyStopTime_auto]");
	var timeInHour = target.find("input[name=timeInHour]");
	var timeInMinute = target.find("input[name=timeInMinute]");
	var timeOutHour = target.find("input[name=timeOutHour]");
	var timeOutMinute = target.find("input[name=timeOutMinute]");
	var startTankPressure = target.find("input[name=startTankPressure]");
	var endTankPressure = target.find("input[name=endTankPressure]");	
	var divePlanningTool = target.find("select[name=divePlanningTool]");
	var weight = target.find("input[name=weight]");
	var enrichedAirBlend = target.find("input[name=enrichedAirBlend]");
	var suitType = target.find("select[name=suitType]");
	var hood = target.find("input[name=hood]");
	var gloves = target.find("input[name=gloves]");
	var boots = target.find("input[name=boots]");	
	var visibility = target.find("input[name=visibility]");
	var diveType = target.find("select[name=diveType]");
	var waterType = target.find("select[name=waterType]");
	var waveType = target.find("select[name=waveType]");
	var currentType = target.find("select[name=currentType]");
	var maxTemperature = target.find("input[name=maxTemperature]");
	var minTemperature = target.find("input[name=minTemperature]");	
	var activity = target.find("input[name=activity]");	
	
	location.val(dive.location);
	point.val(dive.point);	
	
	if(dive.surfaceIntervalHour > 0 || dive.surfaceIntervalMinute > 0) {
		surfaceIntervalHour.val(dive.surfaceIntervalHour);
		surfaceIntervalMinute.val(dive.surfaceIntervalMinute);
	}
	
	if(dive.enrichedAirBlend > 0)
		enrichedAirBlend.val(dive.enrichedAirBlend);
	
	maxDepth.val(dive.maxDepth);
	averageDepth.val(dive.averageDepth);
	diveTime.val(dive.diveTime);
	safetyStopTime.val(dive.safetyStopTime);	
	timeInHour.val(dive.timeInHour);
	timeInMinute.val(dive.timeInMinute);
	timeOutHour.val(dive.timeOutHour);
	timeOutMinute.val(dive.timeOutMinute);
	startTankPressure.val(dive.startTankPressure);
	endTankPressure.val(dive.endTankPressure);
	visibility.val(dive.visibility);
	waveType.val(dive.waveType);
	currentType.val(dive.currentType);
	maxTemperature.val(dive.maxTemperature);
	minTemperature.val(dive.minTemperature);
	activity.val(dive.activity);
	
	if(type == 'copy')
		return;
	
	var note = target.find("textarea[name=note]");	
	var stickers = target.find("input[name=stickers]");
	var signeeId = target.find("input[name=signeeId]");
	var signeeName = target.find("input[name=signeeName]");	
	
	note.val(dive.note);
	stickers.val(dive.stickers);
	signeeId.val(dive.signeeId);
	signeeName.val(dive.signeeName);
	
	setLogbookStickerGeneration(dive.stickers);
	
	var btn_location = '';
	
	if(dive.signatureStatus == "Requested")
		btn_location = "./main/images/btn_buddy_pick_requested.jpg";
	else if(dive.signatureStatus == "Refused")
		btn_location = "./main/images/btn_buddy_pick_refused.jpg";
	else if(dive.signatureStatus == "Put") {
		findSigneeBtn.attr("disabled", true);
		btn_location = "./main/images/btn_buddy_pick_put.jpg";
	}
	
	if(btn_location.length > 0)
		findSigneeBtn.attr("src", btn_location);	
	
	if(type == 'modify')
		return;
	
	// restricted... 
	target.find(".dive_setting").hide();
	
	diveDate.val(dive.diveDate);	
	
	diveNumber.attr('readonly', 'readonly');	
	location.attr('readonly', 'readonly');
	point.attr('readonly', 'readonly');		
	surfaceIntervalHour.attr('readonly', 'readonly');
	surfaceIntervalMinute.attr('readonly', 'readonly');	
	maxDepth.attr('readonly', 'readonly');
	averageDepth.attr('readonly', 'readonly');
	diveTime.attr('readonly', 'readonly');
	safetyStopTime.attr('readonly', 'readonly');
	safetyStopTime_auto.attr('readonly', 'readonly');
	timeInHour.attr('readonly', 'readonly');
	timeInMinute.attr('readonly', 'readonly');
	timeOutHour.attr('readonly', 'readonly');
	timeOutMinute.attr('readonly', 'readonly');
	startTankPressure.attr('readonly', 'readonly');
	endTankPressure.attr('readonly', 'readonly');	
	weight.attr('readonly', 'readonly');
	enrichedAirBlend.attr('readonly', 'readonly');	
	visibility.attr('readonly', 'readonly');	
	maxTemperature.attr('readonly', 'readonly');
	minTemperature	.attr('readonly', 'readonly');
	activity	.attr('readonly', 'readonly');	
	
	diveDate.attr('disabled', true);
	divePlanningTool.attr("disabled", true);
	suitType.attr("disabled", true);
	hood.prop("disabled", "disabled");
	gloves.prop("disabled", "disabled");
	boots.prop("disabled", "disabled");
	diveType.attr("disabled", true);
	waterType.attr("disabled", true);
	waveType.attr("disabled", true);
	currentType.attr("disabled", true);
	signeeName.attr("disabled", true);
}

function setLogbookProfileView(dive, target) {
	dive.samples = eval("(" + dive.samples + ")");
	
	if(dive.samples == null)
		return;
	
	var profile = parseLogbookProfile(dive);
	
	var data = {
        labels: profile.labels,
        datasets: [
            {
            	label: lang["depth_text"],
            	yAxesGroup: "Depth",
            	fillColor: "rgba(32, 128, 208, 0.2)",
            	strokeColor: "rgba(32, 128, 208, 0.8)",
            	pointColor: "rgba(32, 128, 208, 0.8)",
            	pointStrokeColor: "#fff",
            	pointHighlightFill: "#fff",
            	pointHighlightStroke: "rgba(32, 128, 208, 0.8)",
            	data: profile.depth
            },            
            {
            	label: lang["water_temperature_text"],        
            	yAxesGroup: "Temperature",
            	fillColor: "rgba(220, 71, 62, 0)",
            	strokeColor: "rgba(220, 71, 62, 0.5)",
            	pointColor: "rgba(220, 71, 62, 0.5)",
            	pointStrokeColor: "#fff",
            	pointHighlightFill: "#fff",
            	pointHighlightStroke: "rgba(220, 71, 62, 0.5)",
            	data: profile.temperature
            }            
        ],
        yAxes: [
			{
				name: "Temperature",
				scalePositionLeft: false,
				scaleFontColor: "rgba(220, 71, 62, 0.8)"
			}
        ]        
    }
	
	g_logbookGraph = new Chart(target.get(0).getContext("2d")).Line(data, {
    	responsive: true,
    	pointHitDetectionRadius: 2,
    	pointDotRadius: 1,
    	pointDot: false,
    	multiTooltipTemplate: '<%=datasetLabel%> : <%if(value <= 0){%><%=(value * -1) + getGraphUnit(value)%><%}else{%><%=value + getGraphUnit(value)%><%}%>',
    	scaleLabel: '<%if(value <= 0){%><%=(value * -1) + getGraphUnit(value)%><%}else{%><%=value + getGraphUnit(value)%><%}%>',
    	labelsFilter: function(value, index) {
    		return (index) % (300 / dive.sampleInterval) !== 0;
        }
    });
}

function setLogbookProfileEditView(dive, target) {
	dive.samples = eval("(" + dive.samples + ")");
	
	if(dive.samples == null)
		return;
	
	var profile = parseLogbookProfile(dive);
	
	var data = {
        labels: profile.labels,
        datasets: [
        	{
        		label: "Depth",            
        		fillColor: "rgba(32, 128, 208, 0.2)",
            	strokeColor: "rgba(32, 128, 208, 0.8)",
            	pointColor: "rgba(32, 128, 208, 0.8)",
            	pointStrokeColor: "#fff",
            	pointHighlightFill: "#fff",
            	pointHighlightStroke: "rgba(32, 128, 208, 0.8)",
              	data: profile.depth
            }
        ]
    }
	
	var distanceUnit = dive.distanceUnit == "Metric" ? "m" : "ft";
	
	g_logbookGraph = new Chart(target.get(0).getContext("2d")).Line(data, {
    	responsive: true,
    	//animationEasing: "easeInOutQuart",
    	pointHitDetectionRadius: 0,
    	pointDotRadius: 1,
    	pointDot: false,
    	scaleShowLabels: false,
    	scaleFontSize: 0,
    	multiTooltipTemplate: '<%= value * -1 %>',
    	tooltipTemplate: '<%if(label){%><%=label%><%}%> - <%= value * -1 %>' + distanceUnit,
    	scaleLabel: '<%= value * -1 %>',
    	scaleShowVerticalLines: false,
    	scaleLineColor:"rgba(255, 255, 255, 0)",    	
    	labelsFilter: function(value, index) {
    		return (index + 1) % 30 !== 0;		
        }
    });	
}

function setLogbookProfileDetailView() {
	var dive = g_logbookSelectedItem;
	
	if(dive == null || dive.samples == null)
		return;	
	
	var profile = parseLogbookProfile(dive);	
	
	var view = '';
	
	view += '<div class="layer_popup2 log_graph_view">';
	view +=	'<div id="auto_detail_graph_header">';
	view +=		'<div class="logo logo_middle"><img src="./_base/images/logo_logbook.png"></div>';
	view +=		'<div class="summary">';
	view +=			'<table summary="요약 정보" cellspacing=0>';
	view +=				'<colgroup>';
	view +=					'<col width="100px"/>';
	view +=					'<col width="100px"/>';
	view +=					'<col width="100px"/>';
	view +=					'<col width="100px"/>';
	view +=					'<col width="100px"/>';
	view +=					'<col width="100px"/>';
	view +=				'</colgroup>';
	view +=				'<tbody>';							
	view +=					'<tr>';
	view +=						'<td scope="col"><p>' + lang["si"] + '</p><input type="text" readonly="readonly" value="' + getSurfaceInterval(dive) + '"/></td>';			
	view +=						'<td scope="col"><p>' + lang["max_depth"] + '</p><input type="text" readonly="readonly" value="' + dive.maxDepth + getDistanceUnit(dive.distanceUnit) + '"/></td>';
	view +=						'<td scope="col"><p>' + lang["average_depth"] + '</p><input type="text" readonly="readonly" value="' + dive.averageDepth + getDistanceUnit(dive.distanceUnit) + '" /></td>';
	view +=						'<td scope="col"><p>' + lang["dive_time"] + '</p><input type="text" readonly="readonly" value="' + dive.diveTime + ' min" /></td>';
	view +=						'<td scope="col"><p>' + lang["max_temperature"] + '</p><input type="text" readonly="readonly" value="' + dive.maxTemperature + getTemperatureUnit(dive.temperatureUnit) + '" /></td>';
	view +=						'<td scope="col"><p>' + lang["min_temperature"] + '</p><input type="text" readonly="readonly" value="' + dive.minTemperature + getTemperatureUnit(dive.temperatureUnit) + '" /></td>';
	view +=					'</tr>';											
	view +=				'</tbody>';
	view +=			'</table>';
	view +=		'</div>';
	view +=	'</div>';
	view +=	'<div id="graph_detail_canvas_wrap">';
	view +=		'<canvas id="graph_detail_canvas"></canvas>';
	view +=	'</div>';
	view +=	'<div id="auto_detail_graph_footer">';
	view +=		'<table summary="상세 정보" cellspacing=0>';
	view +=			'<colgroup>';
	view +=				'<col width="95px"/>';
	view +=				'<col width="120px"/>';
	view +=				'<col width="95px"/>';
	view +=				'<col width="120px"/>';
	view +=				'<col width="95px"/>';
	view +=				'<col width="120px"/>';
	view +=				'<col width="95px"/>';
	view +=				'<col width="120px"/>';
	view +=			'</colgroup>';
	view +=			'<tbody>';							
	view +=				'<tr>';
	view +=					'<td scope="col"><p>' + lang["time_text"] + '</p></td>';			
	view +=					'<td scope="col"><input type="text" name="time" readonly="readonly" value="--"/></td>';
	view +=					'<td scope="col"><p>' + lang["depth_text"] + '</p></td>';
	view +=					'<td scope="col"><input type="text" name="depth" readonly="readonly" value="--' + getDistanceUnit(dive.distanceUnit) + '"/></td>';
	view +=					'<td scope="col"><p>' + lang["water_temperature_text"] + '</p></td>';
	view +=					'<td scope="col"><input type="text" name="temperature" readonly="readonly" value="--' + getTemperatureUnit(dive.temperatureUnit) + '"/></td>';
	view +=					'<td scope="col"><p>' + lang["tank_text"] + '</p></td>';
	view +=					'<td scope="col"><input type="text" name="tank" readonly="readonly" value="--' + getPressureUnit(dive.pressureUnit) + '"/></td>';
	view +=				'</tr>';											
	view +=			'</tbody>';
	view +=		'</table>';
	view +=	'</div>';
	view += '</div>';		
		
	var popup = showPopup2(view);
	
	if($(window).height() < 768) {		
		var height = $(window).height() - 40;
		var gap = 400 - (768 - height);
		
		$("#graph_detail_canvas").css("height", gap);
		$("#popup2 .layer_popup2").css("height", height);
	}		
	
	var data = {
        labels: profile.labels,
        datasets: [
            {
            	label: lang["depth_text"],
            	yAxesGroup: "Depth",
            	fillColor: "rgba(32, 128, 208, 0.2)",
            	strokeColor: "rgba(32, 128, 208, 0.8)",
            	pointColor: "rgba(32, 128, 208, 0.8)",
            	pointStrokeColor: "#fff",
            	pointHighlightFill: "#fff",
            	pointHighlightStroke: "rgba(32, 128, 208, 0.8)",
            	data: profile.depth
            },            
            {
            	label: lang["water_temperature_text"],        
            	yAxesGroup: "Temperature",
            	fillColor: "rgba(220, 71, 62, 0)",
            	strokeColor: "rgba(220, 71, 62, 0.5)",
            	pointColor: "rgba(220, 71, 62, 0.5)",
            	pointStrokeColor: "#fff",
            	pointHighlightFill: "#fff",
            	pointHighlightStroke: "rgba(220, 71, 62, 0.5)",
            	data: profile.temperature
            }            
        ],
        yAxes: [
			{
				name: "Temperature",
				scalePositionLeft: false,
				scaleFontColor: "rgba(220, 71, 62, 0.8)"
			}
        ]        
    }
	
	g_logbookDetailGraph = new Chart($("#graph_detail_canvas").get(0).getContext("2d")).Line(data, {
    	responsive: true,
    	pointHitDetectionRadius: 2,
    	pointDotRadius: 1,
    	pointDot: false,
    	multiTooltipTemplate: '<%=datasetLabel%> : <%if(value <= 0){%><%=(value * -1) + getGraphUnit(value)%><%}else{%><%=value + getGraphUnit(value)%><%}%>',
    	scaleLabel: '<%if(value <= 0){%><%=(value * -1) + getGraphUnit(value)%><%}else{%><%=value + getGraphUnit(value)%><%}%>',
    	labelsFilter: function(value, index) {
    		return (index) % (300 / dive.sampleInterval) !== 0;
        }
    });
}

function getGraphUnit(value) {
	var unit = "";
	
	if(value <= 0)
		unit = g_logbookSelectedItem.distanceUnit == "Metric" ? "m" : "ft";
	else
		unit = g_logbookSelectedItem.temperatureUnit == "Celsius" ? "℃" : "℉";
	
	return unit;
}

function setLogbookVideoLinkView() {
	var view = '';
	
	view += '<div class="layer_popup upload_movie">';
	view += 	'<div class="contain">';
	view += 		'<h1>' + lang["video_link"] + '</h1>';
	view +=		'<form name="logbook_video_link_form">';
	view += 			'<div class="uri_embed">';
	view += 				'<div class="messeage">' + lang["video_link_inst_input"] + '</div>';
	view += 				'<div class="link"><input type="text" name="link"/></div>';
	view += 				'<div class="notify">' + lang["video_link_inst_warning"] + '</div>';
	view += 			'</div>';
	view += 			'<div class="function">';
	view += 				'<div class="bside">';
	view += 					'<div class="submit">';
	view += 						'<input type="submit" value="' + lang["confirm"] + '" />&nbsp;';
	view += 						'<input type="reset" name="logbook_cancel_video_link_btn" value="' + lang["cancel"] + '" />';
	view += 					'</div>';
	view += 				'</div>';
	view += 			'</div>';
	view +=		'</form>';
	view += 	'</div>';
	view += '</div>';
	
	showPopup(view);	
}

function setLogbookGalleryView() {
	if(g_logbookGalleryView.length == 0)
		return;
	
	showPopup2(g_logbookGalleryView);	
	$(".cycle-slideshow").cycle();
	
	if($(window).height() < 768) {
		$(".photo_view").css("height", $(window).height() - 40);
		$(".photo_book .showcase_image").css("max-height", $(window).height() - 130);
		$(".photo_book .showcase_image li img").css("max-height", $(window).height() - 130);
	}
}

function setLogbookStickerView(stickers) {
	g_logbookSelectedSticker = "";
		
	if(stickers != null) {
		var type1 = '';
		var type2 = '';
		var type3 = '';
		var type4 = '';
		
		// categorizing...		
		var view = '';
		
		$(stickers).each(function(index, sticker) {
			view = '<li><a href="' + sticker.id + '"><img src="./main/images/stickers/' + sticker.id + '.png"/></a></li>';
			
			if(sticker.category == "Type1")
				type1 += view;
			else if(sticker.category == "Type2")
				type2 += view;
			else if(sticker.category == "Type3")
				type3 += view;
			else
				type4 += view;
		});
		
		// generate...		
		g_logbookStickerView	= '';
		
		g_logbookStickerView += '<div class="layer_popup sticker">';
		g_logbookStickerView += 	'<div class="contain">';
		g_logbookStickerView += 		'<div class="category">';
		g_logbookStickerView += 			'<div class="list cycle-slideshow" data-cycle-timeout="0" data-cycle-slides="> div" data-cycle-fx="scrollHorz" data-allow-wrap="false" data-cycle-prev=".category .prev" data-cycle-next=".category .next">';
		g_logbookStickerView +=				'<div>';
		g_logbookStickerView += 					'<a href="#" onclick="$(\'.emoticon .list\').cycle(\'goto\', 0)" class="type1 active">type1</a>';
		g_logbookStickerView += 					'<a href="#" onclick="$(\'.emoticon .list\').cycle(\'goto\', 1)" class="type2">type2</a>';
		g_logbookStickerView += 					'<a href="#" onclick="$(\'.emoticon .list\').cycle(\'goto\', 2)" class="type3">type3</a>';
		g_logbookStickerView += 					'<a href="#" onclick="$(\'.emoticon .list\').cycle(\'goto\', 3)" class="type4">type4</a>';
		g_logbookStickerView += 				'</div>';	
		g_logbookStickerView +=			'</div>';
		g_logbookStickerView += 			'<button class="prev">prev</button>';
		g_logbookStickerView += 			'<button class="next">next</button>';
		g_logbookStickerView +=		'</div>';
		g_logbookStickerView += 		'<div class="emoticon">';
		g_logbookStickerView += 			'<div class="list cycle-slideshow" data-cycle-timeout="0" data-cycle-slides="> ul" data-cycle-fx="scrollHorz">';
		g_logbookStickerView +=				'<ul>' + type1 + '</ul>';
		g_logbookStickerView +=				'<ul>' + type2 + '</ul>';
		g_logbookStickerView +=				'<ul>' + type3 + '</ul>';
		g_logbookStickerView +=				'<ul>' + type4 + '</ul>';
		g_logbookStickerView += 			'</div>';
		g_logbookStickerView += 		'</div>';
		g_logbookStickerView += 		'<div class="function">';
		g_logbookStickerView += 			'<div class="submit">';
		g_logbookStickerView += 				'<input type="submit" name="logbook_add_sticker_btn" value="' + lang["append"] + '" />&nbsp;';
		g_logbookStickerView += 				'<input type="reset" name="logbook_cancel_add_sticker_btn" value="' + lang["cancel"] + '" />';
		g_logbookStickerView += 			'</div>';
		g_logbookStickerView += 		'</div>';
		g_logbookStickerView += 	'</div>';
		g_logbookStickerView += '</div>';
	}

	showPopup(g_logbookStickerView);
	$(".cycle-slideshow").cycle();
}

function setLogbookAddSticker(sticker) {
	hidePopup();
	
	var id = getLogbookStickerId(sticker);
	
	if(getLogbookStickerObject(sticker).length > 0) {
		showMessage(lang["sticker_exist"]);
		return;
	}
	
	var view = '';
		
	view += '<li id="' + id + '">';
	view += 	'<div class="emoticon">';
	view += 		'<img src="./main/images/stickers/' + sticker + '.png" />';
	view += 		'<button type="button" name="' + id + '">X</button>';
	view += 	'</div>';
	view += '</li>';
	
	$(".sticker_list ul").append(view);
}

function setLogbookStickerGeneration(stickers) {
	if(stickers == null || stickers.length == 0)
		return;
	
	var id = '';
	var view = '';
	
	$(stickers.split(";")).each(function(index, sticker) {
		if(sticker == null || sticker.length == 0)
			return true;
		
		id = getLogbookStickerId(sticker);
		
		view += '<li id="' + id + '">';
		view += 	'<div class="emoticon">';
		view += 		'<img src="./main/images/stickers/' + sticker + '.png" />';
		view += 		'<button type="button" name="' + id + '">X</button>';
		view += 	'</div>';
		view += '</li>';
	});
	
	$(".sticker_list ul").append(view);
}

function setLogbookPageView(count) {
	g_logbookTotalPageCount = setItemPageViewInit(count, g_logbookPageSize);	
	setLogbookPaging(g_logbookCurrentPageNumber, g_logbookTotalPageCount);
}

function setLogbookPaging(pageNumber, pageCount) {
	g_logbookCurrentPageNumber = pageNumber;
	setItemPaging($("#news_feed .post_pager"), "logbook_page_btn", "", pageNumber, pageCount);
}

function setLogbookLikeCount(userId, logbookNumber, count) {
	var countBtn = getLogbookObject(userId, logbookNumber).find(".logbook_like_btn").find("b");
	
	countBtn.empty();
	countBtn.append(count);
	
	if(isVisiblePopup()) {
		id = '#logbook__' + userId + '__' + logbookNumber;
		countBtn = $(id).find(".logbook_like_btn").find("b");
	
		countBtn.empty();
		countBtn.append(count);
	}
}

function setLogbookSigneeInfo() {
	var buddyId = '';
	var buddyName = '';
	
	$("input[name=buddy_select]").each(function(index, item) {
		var checked = $(item).prop("checked");
		if(checked)
		{
			var info = $(item).val().split("__");
			buddyId = info[0];
			buddyName = info[1];
		}
	});	
	
	if(buddyId == "" || buddyName == "")
		return;	
	
	$("input[name=signeeId]").val(buddyId);
	$("input[name=signeeName]").val(buddyName);	
}

function setLogbookBulkImportResult(index, result, message, command) {
	var item = $("#logbook_import_file_" + index);
	
	item.find(".uploading_btn").hide();
	item.find(".right").empty();
	
	if(result)
		message = lang["succeeded"];
	else
		message = lang["failed"] + " - " + message;
	
	item.find(".right").append(message);
	item.find(".right").attr("title", message);
	
	if(++g_logbookImportedCount == g_logbookImportTotalCount) {
		showMessage(lang["completed_upload"]);
		initLogbookBulkImportChooseFilesView();	
	}
	else
		readLogFiles("Bulk", ++index, command);
}

function getCheckedExposureProtection(dive, item) {
	if(item == "hood")
		return dive.hood == true ? "checked" : "";
	
	if(item == "gloves")
		return dive.gloves == true ? "checked" : "";
	
	if(item == "boots")
		return dive.boots == true ? "checked" : "";
	
	return "";
}

function getSurfaceInterval(dive) {
	if(dive.surfaceIntervalHour == 0 && dive.surfaceIntervalMinute == 0)
		return " : ";
	
	return getFormatedNumber(dive.surfaceIntervalHour) + ':' + getFormatedNumber(dive.surfaceIntervalMinute);
}

function getTime(hour, minute) {
	return getFormatedNumber(hour) + ':' + getFormatedNumber(minute);
}

function getDistanceUnit(unit) {
	return unit == "Metric" ? " m" : " ft";
}

function getWeightUnit(unit) {
	return unit == "Kilogram" ? " kg" : " lbs";
}

function getTemperatureUnit(unit) {
	return unit == "Celsius" ? " ℃" : " ℉";
}

function getPressureUnit(unit) {
	return unit == "Bar" ? " bar" : " psi";
}

function getEnrichedAirBlend(dive) {
	if(dive.enrichedAirBlend == 0)		
		return " - ";
	
	return dive.enrichedAirBlend + " %";	
}

function getDivePlanningTool(tool) {
	if(tool == "Computer")
		return lang["planning_tool_computer"];
	
	if(tool == "Table")
		return lang["planning_tool_table"];
	
	return lang["planning_tool_other"];
}

function getExposureProtection(dive) {
	var result = '';
	
	if(dive.suitType == "Skin")
		result += lang["suit_type_skin"];
	else if(dive.suitType == "Wet")
		result += lang["suit_type_wet"];
	else	
		result += lang["suit_type_dry"];
	
	if(dive.hood)
		result += ', ' + lang["hood"];
	
	if(dive.gloves)
		result += ', ' + lang["gloves"];
	
	if(dive.boots)
		result += ', ' + lang["boots"];
	
	return result;
}

function getDiveType(dive) {
	return dive.diveType == "Boat" ? lang["dive_type_boat"] : lang["dive_type_shore"];
}

function getWaterType(dive) {
	return dive.waterType == "Salt" ? lang["water_type_salt"] : lang["water_type_fresh"];
}

function getStrengthType(type) {
	if(type == "Strong")
		return lang["strength_strong"];
	
	if(type == "Normal")
		return lang["strength_normal"];
	
	return lang["strength_none"];
}

function getLogbookGalleryView(fileList) {
	if(fileList.length == 0)
		return '';
	
	// source view...
	var sourceView = '';
	
	$(fileList).each(function(index, file) {
		if(file.fileType == "Picture")
			sourceView += '<li><div class="align"><img src="' + file.sourceLocation + '"/></div></li>';
		else if(file.fileType == "VideoLink")
			sourceView += '<li><div class="align">' + file.sourceLocation + '</div></li>';
	});	
	
	// thumbnail view...
	var thumbnailView = '';
	var count = 0;
	var totalCount = 0;
	
	$(fileList).each(function(index, file) {
		count++;
		totalCount++;
		
		if(count == 1)
			thumbnailView += '<div>';
		
		if(totalCount == 1)
			thumbnailView += '<a href="' + index + '" class="active"><img src="' + file.thumbnailLocation + '" /></a>';			
		else
			thumbnailView += '<a href="' + index + '"><img src="' + file.thumbnailLocation + '" /></a>';
		
		if(count == 15) {
			thumbnailView += '</div>';
			count = 0;
		}
	});
	
	if((totalCount % 15) != 0)
		thumbnailView += '</div>';
	
	// integrated view...	
	var view = '';
	
	view += '<div class="layer_popup2 photo_view">';
	view += 	'<div class="photo_book" id="logbook_photo_view">';
	view += 		'<div class="showcase_image">';
	view +=			'<ul class="list cycle-slideshow" data-cycle-timeout="0" data-cycle-fx="scrollHorz" data-cycle-slides="> li" data-allow-wrap="false">';
	view +=				sourceView;
	view +=			'</ul>';
	view +=			'<div class="prev"><button><img src="./main/images/btn_showcase_prev.png" /></button></div>';
	view +=			'<div class="next"><button><img src="./main/images/btn_showcase_next.png" /></button></div>';
	view +=		'</div>';
	view +=		'<div class="showcase_thumb">';
	view +=			'<div class="list cycle-slideshow" data-cycle-timeout="0" data-cycle-fx="scrollHorz" data-cycle-slides="> div" data-allow-wrap="false">';
	view +=				thumbnailView;
	view +=			'</div>';
	view +=			'<div class="pager">';
	view +=				'<button class="prev"><img src="./main/images/btn_showcase_prev.png" /></button>';
	view +=				'<button class="next"><img src="./main/images/btn_showcase_next.png" /></button>';
	view +=			'</div>';
	view +=		'</div>';
	view += 	'</div>';
	view += '</div>';
	
	return view;	
}

function getStickers(stickers) {
	if(stickers == null || stickers.length == 0)
		return '';
	
	var view = '';
	
	$(stickers.split(";")).each(function(index, sticker) {
		if(sticker == null || sticker.length == 0)
			return true;
		
		view += '<img src="./main/images/stickers/' + sticker + '.png"/>';
	});
	
	return view;
}

function getSignature(dive) {
	var signature= '';
	var command = '';
	
	if(dive.signatureStatus != "Put") {
		signature = dive.signeeName;
		
		if(dive.signatureStatus == "Requested" && dive.signeeId == g_myId) {
			var id = getLogbookId(dive.userId, dive.logbookNumber);
			
			command += '<div class="action">';
			command += 	'<button class="logbook_put_signature_btn verification" name="' + id + '">' + lang["signature"] + '</button>&nbsp;';
			command += 	'<button class="logbook_refuse_signature_btn" name="' + id + '">' + lang["refuse"] + '</button>';
			command += '</div>';
		}		
	}
	else
		signature = getText(dive.signature);
	
	var view = '';
	
	view += '<div>';
	view +=	'<div class="sign">';
	view +=		'<h1 class="title">' + lang["signature"] + '</h1>';
	view +=		'<div class="ellipsis">';
	view +=			'<div>';
	view +=				'<a class="user_link_dir" href="' + dive.signeeId + '">' + signature + '</a>';
	view +=			'</div>';
	view +=		'</div>';
	view +=	'</div>';
	view +=	'<div class="mark">';
	view +=		'<a class="user_link_dir" href="' + dive.signeeId + '"><img src="' + getSignatureImage(dive) + '" /></a>';
	view +=	'</div>';
	view += 	command;
	view += '</div>';
	
	return view;
}

function getSignatureImage(dive) {
	var status = dive.signatureStatus;
	
	if(status == "Requested")
		return "./main/images/logbook_requested.jpg";
	else if(status == "Refused")
		return "./main/images/logbook_refused.jpg";
	else if(status == "Put")
		return dive.stampLocation;
	
	return "./main/images/logbook_not_requested.jpg";
}

function getSignatureEditImage(type, dive) {
	if(type == "new" || type == "copy")
		return "./main/images/btn_buddy_pick_not_requested.jpg";
	
	var status = dive.signatureStatus;
	
	if(status == "Requested")
		return "./main/images/btn_buddy_pick_requested.jpg";
	else if(status == "Refused")
		return "./main/images/btn_buddy_pick_refused.jpg";
	else if(status == "Put")
		return "./main/images/btn_buddy_pick_put.jpg";
	
	return "./main/images/btn_buddy_pick_not_requested.jpg";	
}

function getLogbookDiveInfo(id, location, point) {
	if(point.length > 0)
		location += ' - ' + point;
	
	return '<a href="' + id + '" class="logbook_body">' + location + '</a>';		
}

function getLogbookItemView(dive, showImages) {
	var id = getLogbookId(dive.userId, dive.logbookNumber);	
	
	var ad = '';
	var adData = '';
	if(dive.isPostedAd) {
		ad = '<span class="ad"><img src="./main/images/icon_ad.png" /></span>';
		adData = ' data-ad="' + dive.adNumber + '__' + dive.contentItemNumber + '"';
	}
	
	var location = dive.location;
	if(dive.point.length > 0)
		location += ' - ' + dive.point;	
	
	var thumbnailImages = '';
	if(showImages)
		thumbnailImages = getLogbookThumbnailImageOneView(dive, adData);
	
	var view = '';
	
	view += '<div class="post" id="' + id + '"' + adData + '>';
	view += 	thumbnailImages;
	view += 	'<div class="user">';
	view += 		'<div class="avatar user_link"><a href="' + dive.userId + '"><img src="' + dive.userImageLocation + '"/></a></div>';
	view += 		'<div class="info">';
	view += 			'<div class="nickname">';
	view +=				ad;
	view +=				'<a class="user_link_dir" href="' + dive.userId + '">' + dive.userName + '</a> ';
	view +=				'<span class="dive_no">Dive No. <b>' + dive.diveNumber + '</b></span>';
	view +=			'</div>';
	view += 			'<div class="date">' + dive.diveDate + '</div>';
	view += 		'</div>';
	view += 	'</div>';
	view += 	'<div class="comments logbook_body">';
	view += 		'<a href="' + id + '"' + adData + '>' + location + '</a>';
	view += 	'</div>';	
	//view += 	thumbnailImages;
	view += 	'<div class="control">';
	view += 		'<div class="response">';
	view += 			'<a class="logbook_like_btn" href="' + id + '">' + lang["like"] + ' <b>' + dive.likeCount + '</b></a>';
	view += 			'<a class="logbook_comment_btn" href="' + id + '"' + adData + '>' + lang["comment"] + ' <b>' + dive.commentCount + '</b></a>';
	view += 		'</div>';
	view +=		getLogbookItemMenu(dive, false);
	view += 	'</div>';
	view += '</div>';

	return view;
}

function getLogbookThumbnailImageView(dive, adData, detail) {
	if(dive.fileList.length == 0)
		return '';
	
	var itemId = getLogbookId(dive.userId, dive.logbookNumber);
	var className = detail ? "logbook_thumbnail_image_detail_item" : "logbook_thumbnail_image_item";
	
	var id = '';
	var view = '';
	
	view += '<div class="album">';
	view += 	'<ul>';
	
	$(dive.fileList).each(function(index, file) {
		id = itemId + '__' + file.itemNumber + '__image';		
		
		if(!detail && index == 8)
			return false;
		
		if(detail && index == 8) {
			view += '<li id="' + id + '"><button class="' + className + '">more</button></li>';
			return false;
		}
		
		view += '<li id="' + id + '"' + adData + '><a href="#" class="' + className + '"><img src="' + file.thumbnailLocation + '"/></a></li>';
	});
	
	view += 	'</ul>';
	view += '</div>';
	
	return view;
}

function getLogbookThumbnailImageOneView(dive, adData) {
	var index = getRandomOne(dive.fileList);
	
	if(index < 0)
		return '';	
	
	var itemId = getLogbookId(dive.userId, dive.logbookNumber);
	
	var id = itemId + '__' + dive.fileList[index].itemNumber + '__image';		
	var view = '';
	
	view += '<div id="' + id + '"' + adData + '>';
	view += 	'<a href="#" class="logbook_thumbnail_image_item">';
	view +=		'<div class="right_image">';
	view +=			'<img src="' + dive.fileList[index].thumbnailLocation + '"/>'
	view +=		'</div>';
	view +=		'<div class="album">';
	view +=			'<b>' + dive.fileList.length + '</b>';
	view +=		'</div>';
	view +=	'</a>';
	view += '</div>';	
	
	return view;
}

function getLogbookItemMenu(dive, detailView) {
	if(dive.isPostedAd)
		return '';
	
	var view = '';	
	var id = getLogbookId(dive.userId, dive.logbookNumber);	

	var modify = detailView ? '' : ' modify';
	var remove = detailView ? '' : ' delete';
	var url = detailView ? '' : ' url';
	
	view += '<div class="action">';
	
	if(dive.userId  == g_myId) {
		view += '<a class="logbook_export_facebook_btn" href="' + id + '"><img src="./main/images/ico_facebook.png" /></a>';
		view += '<button class="logbook_modify_btn' + modify + '" name="' + id + '">' + lang["modify"] + '</button>';
		
		if(!dive.isAd)
			view += '<button class="logbook_delete_btn' + remove + '" name="' + id + '">' + lang["remove"] + '</button>';
		
		view += '<button class="logbook_url_btn' + url + '" name="' + id + '">' + lang["copy_url"] + '</button>';		
	}
	
	if(detailView && (dive.buddyStatus == "Accepted" || dive.buddyStatus == "Favorite"))
		view += '<button class="logbook_copy_btn" name="' + id + '">' + lang["copy_log"] + '</button>';	
	
	if(detailView && g_myId != null && dive.buddyStatus == "NotBuddy")
		view += '<button class="logbook_buddy_requeset_btn" name="' + id + '"><b>' + lang["request_buddy"] + '</b></button>';
	
	view += '</div>';
	
	return view;
}

function getLogbookUploadedFilesView(type, dive, itemId) {
	if(type == "new" || type == "copy")
		return '';
	
	if(dive.fileList.length == 0)
		return '';
	
	var view = '';
	var id = '';
	
	view += '<div class="uploaded_list">';
	view += 	'<ul>';
	
	$(dive.fileList).each(function(index, file) {		
		id = itemId + '__' + file.itemNumber + '__image';
		
		view += '<li>';
		view += 	'<div class="thumb">';
		view += 		'<img src="' + file.thumbnailLocation + '" />';
		view +=		'<button type="button" id="' + id + '" class="logbook_thumbnail_image_remove_btn">X</button>';
		view +=	'</div>';
		view += '</li>';				
	});
	
	view += 	'</ul>';
	view += '</div>';
	
	return view;	
}

function setLogbookGraphValue(event) {
	var items = g_logbookGraph.getPointsAtEvent(event);
	
	if(items.length == 0)
		return;
	
	var dive = g_logbookSelectedItem;
	
	var distanceUnit = dive.distanceUnit == "Metric" ? " m" : " ft"
	var temperatureUnit = dive.temperatureUnit == "Celsius" ? " ℃" : " ℉";
	var pressureUnit = dive.pressureUnit == "Bar" ? " bar" : " psi";
	
	var depth = items[0].value * -1 + distanceUnit;
	var temperature = items[1].value + temperatureUnit;
	
	if(items[1].value < 0) {				
		if(items.length > 2 && items[2].value > -1)
			temperature =  items[2].value + temperatureUnit;		// 화면이 좁은 경우 2개씩 데이터가 넘어오는 경우가 생김
		else
			temperature = "";
	}
	
	$("#graph_time_value").html(items[0].label);
	$("#graph_distance_value").html(depth);
	
	if(temperature.length > 0)
		$("#graph_temperature_value").html(temperature);		
	
	// 개선 필요...
	if(dive.usedTransmitter) {
		var label = "";		
		$(dive.samples).each(function(index, sample) {
			if(sample.tp == null)
				return true;
			
			label = Math.floor(sample.tm / 60) + '\'' + sample.tm % 60;
			
			if(label == items[0].label) {				
				var tankPressure = 0;
				
				if(dive.pressureUnit == "Bar")
					tankPressure = roundOff(Pa_Bar(sample.tp), 0);
				else
					tankPressure = roundOff(Pa_Psi(sample.tp), 0);
				
				$("#graph_pressure_value").html(tankPressure + pressureUnit);
				
				return false;
			}
		});
	}
}


function setLogbookGraphDetailValue(event) {
	var items = g_logbookDetailGraph.getPointsAtEvent(event);
	
	if(items.length == 0)
		return;
	
	var dive = g_logbookSelectedItem;
	
	var distanceUnit = dive.distanceUnit == "Metric" ? " m" : " ft"
	var temperatureUnit = dive.temperatureUnit == "Celsius" ? " ℃" : " ℉";
	var pressureUnit = dive.pressureUnit == "Bar" ? " bar" : " psi";
	
	var depth = items[0].value * -1 + distanceUnit;
	var temperature = items[1].value + temperatureUnit;
	
	$("#auto_detail_graph_footer input[name=time]").val(items[0].label);
	$("#auto_detail_graph_footer input[name=depth]").val(depth);
	$("#auto_detail_graph_footer input[name=temperature]").val(temperature);		
	
	// 개선 필요...
	if(dive.usedTransmitter) {
		var label = "";		
		$(dive.samples).each(function(index, sample) {
			if(sample.tp == null)
				return true;
			
			label = Math.floor(sample.tm / 60) + '\'' + sample.tm % 60;
			
			if(label == items[0].label) {				
				var tankPressure = 0;
				
				if(dive.pressureUnit == "Bar")
					tankPressure = roundOff(Pa_Bar(sample.tp), 0);
				else
					tankPressure = roundOff(Pa_Psi(sample.tp), 0);
				
				$("#auto_detail_graph_footer input[name=tank]").val(tankPressure + pressureUnit);
				
				return false;
			}
		});
	}
}


////////////////////////////////////////////////////////////////////////////////////////////
/// Event Handlers
////////////////////////////////////////////////////////////////////////////////////////////

$(document).ready(function() {
	$(document).on("submit", "form[name=logbook_search_form]", function(event) {
		g_logbookCurrentPageNumber = 1;
		requestLogbookList(g_userId, $(this).serialize(), g_logbookCurrentPageNumber);
		requestLogbookCount(g_userId, $(this).serialize());
		return false;
	});
		
	$(document).on("submit", "form[name=logbook_write_form]", function(event) {
		if(checkLogbook($("form[name=logbook_write_form]"))) {
			var format = $("#datepicker_logbook").datepicker("option", "dateFormat");
			
			if(format != "yy-mm-dd")
				$("#datepicker_logbook").datepicker("option", "dateFormat", "yy-mm-dd");
			
			generateLogbookStickers();			
			var data = $(this).serialize();
			
			if($("select[name=viewAuthority]").val() == "Private" && $("input[name=signeeId]").val() != "") {
				setAlertMessage(lang["confirm_unable_signature"]);
				$("#alert").dialog({
					resizable: false,
					height: 140,
					modal: true,
					buttons: [
			            {
			            	text: lang["yes"],
			            	click: function() {
			            		$(this).dialog("close");					
			            		requestLogbookWrite(data);
			            	}
			            },	            
			            {
			            	text: lang["no"],
			            	click: function() {
			            		$(this).dialog("close");
			            	}
			            }
			        ]		
				});				
			}
			else
				requestLogbookWrite(data);
		}		
		
		return false;
	});
	
	$(document).on("click", "button[name=logbook_bulk_upload_btn]", function(event) {
		var files = document.getElementById("log_files").files;
		
		if(getSelectedImportFileCount(files) == 0) {
			showMessage(lang["logbook_import_select_file"]);
			return false;
		}
		
		requestLogbookTemplateForImport();
		
		return false;
	});
	
	$(document).on("click", "button[name=logbook_sort_btn]", function(event) {
		setAlertMessage(lang["confirm_sort_logbook"]);
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: lang["yes"],
	            	click: function() {
	            		$(this).dialog("close");					
	            		requestLogbookSort();
	            	}
	            },	            
	            {
	            	text: lang["no"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]		
		});
		
		return false;
	});	
		
	$(document).on("submit", "form[name=logbook_modify_form]", function(event) {
		var inputType = $("form[name=logbook_modify_form] input[name=inputType]").val();
		
		if(inputType == "Imported") {
			var safetyStopTime_auto = $("form[name=logbook_modify_form] input[name=safetyStopTime_auto]").val();
			$("form[name=logbook_modify_form] input[name=safetyStopTime]").val(safetyStopTime_auto);
		}			
		
		if(checkLogbook($("form[name=logbook_modify_form]"))) {
			var format = $("#datepicker_logbook").datepicker("option", "dateFormat");
			
			if(format != "yy-mm-dd")
				$("#datepicker_logbook").datepicker("option", "dateFormat", "yy-mm-dd");
			
			generateLogbookStickers();
			var info = $(this).attr("action").split("__");
			var data = $(this).serialize();
			
			if($("select[name=viewAuthority]").val() == "Private" && $("input[name=signeeId]").val() != "") {
				setAlertMessage(lang["confirm_unable_signature"]);
				$("#alert").dialog({
					resizable: false,
					height: 140,
					modal: true,
					buttons: [
			            {
			            	text: lang["yes"],
			            	click: function() {
			            		$(this).dialog("close");					
			            		requestLogbookModify(info[1], info[2], data);
			            	}
			            },	            
			            {
			            	text: lang["no"],
			            	click: function() {
			            		$(this).dialog("close");
			            	}
			            }
			        ]							
				});				
			}
			else
				requestLogbookModify(info[1], info[2], data);			
		}
		
		return false;
	});
	
	$(document).on("submit", "form[name=logbook_modify_restricted_form]", function(event) {
		if(checkLogbookRestricted($("form[name=logbook_modify_restricted_form]"))) {
			generateLogbookStickers();
			var info = $(this).attr("action").split("__");
			requestLogbookModifyRestricted(info[1], info[2], $(this).serialize());
		}
		
		return false;
	});	
	
	$(document).on("submit", "form[name=logbook_video_link_form]", function(event) {
		if(checkVideoLink($("form[name=logbook_video_link_form] input[name=link]").val())) {
			addLogbookVideoLink($(this).find("input[name=link]").val());
			hidePopup();
		}
		
		return false;
	});
	
	$(document).on("click", "input[name=logbook_cancel_video_link_btn]", function(event) {
		hidePopup();		
	});	
	
	$(document).on("click", "button[name=logbook_remove_video_link_btn]", function(event) {
		$(this).parent().parent().remove();
	});
	
	$(document).on("click", ".logbook_page_btn", function(event) {
		if($(this).hasClass("ap_no_active")) {
			return false;
		}	
	
		var pageNumber = getHref($(this).attr("href"));
		
		if(g_mainSelectedMenu == "logbook")
			requestLogbookList(g_userId, $("form[name=logbook_search_form]").serialize(), pageNumber);
		else if(g_mainSelectedMenu == "search")
			requestSearchLogbookList($("input[name=integrated_condition]").val(), pageNumber);
		
		$(window).scrollTop(0);
			
		return false;
	});
	
	$(document).on("click", ".logbook_body", function(event) {
		var info = getHref($(this).find("a").attr("href")).split("__");
		var data = $(this).find("a").attr("data-ad");
				
		if(data != null) {
			g_logbookIsPostedAd = true;
			data = data.split("__");
			requestAdClickContent("Content_Logbook", data[0], data[1]);
		}
		else
			g_logbookIsPostedAd = false;
		
		requestLogbookDetail(info[1], info[2], false);		
		
		return false;
	});	
	
	$(document).on("click", ".logbook_popular", function(event) {
		var info = getHref($(this).attr("href")).split("__");
		
		g_logbookIsPostedAd = false;
		requestLogbookDetail(info[1], info[2], false);
		
		return false;
	});
	
	$(document).on("click", ".logbook_write_btn", function(event) {
		handleLogbookMenu("logbook_write_view");
		return false;
	});
	
	$(document).on("click", ".logbook_copy_btn", function(event) {
		var info = $(this).attr("name").split("__");
		initLogbookCopiedWriteView(info[1], info[2]);		
		return false;
	});
	
	$(document).on("click", ".logbook_bulk_import_btn", function(event) {
		handleLogbookMenu("logbook_bulk_import_view");
		return false;
	});
	
	$(document).on("click", ".logbook_buddy_requeset_btn", function(event) {
		var info = $(this).attr("name").split("__");
		requestBuddyRequestByContent(info[1], $(this));
		return false;
	});	
	
	$(document).on("click", ".logbook_modify_btn", function(event) {
		var info = $(this).attr("name").split("__");
		
		g_logbookIsPostedAd = false;
		requestLogbookDetail(info[1], info[2], true);
		
		return false;
	});
	
	$(document).on("click", ".logbook_export_facebook_btn", function(event) {
		var info = getHref($(this).attr("href")).split("__");	
		requestLogbookExport("Facebook", info[1], info[2]);
		return false;
	});
	
	$(document).on("click", ".logbook_pdf_btn", function(event) {
		var info = getHref($(this).attr("href")).split("__");
		requestLogbookPdfConversion(info[1], info[2]);
		return false;
	});	
	
	$(document).on("click", ".logbook_delete_btn", function(event) {
		var info = $(this).attr("name").split("__");
		
		setAlertMessage(lang["confirm_remove_log"]);
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: lang["yes"],
	            	click: function() {
	            		$(this).dialog("close");					
	            		requestLogbookDelete(info[1], info[2]);	
	            	}
	            },	            
	            {
	            	text: lang["no"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]
		});
		
		return false;
	});
	
	$(document).on("click", ".logbook_url_btn", function(event) {
		var info = $(this).attr("name").split("__");
		var link = makeLink("logbook", info[1], info[2]);
		var message = '';
		
		message += lang["instruct_copied_url"];
		message += '<textarea style="width:360px; height:55px;" onFocus="this.select();">' + link + '</textarea>';
				
		setAlertMessage(message);
		$("#alert").dialog({
			resizable: false,
			width: 400,
			height: 200,
			modal: true,
			buttons: [
	            {
	            	text: lang["confirm"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]				
		});
		
		return false;
	});
		
	$(document).on("click", ".logbook_like_btn", function(event) {
		var info = getHref($(this).attr("href")).split("__");
		requestLogbookLike(info[1], info[2]);
		return false;
	});	
	
	$(document).on("click", ".logbook_view_count_btn", function(event) {
		var info = getHref($(this).attr("href")).split("__");
		
		g_logbookIsPostedAd = false;
		requestLogbookDetail(info[1], info[2], false);
		
		return false;
	});
	
	$(document).on("click", ".logbook_put_signature_btn", function(event) {
		var info = $(this).attr("name").split("__");
		requestLogbookPutSignature(info[1], info[2], false);		
		return false;
	});
	
	$(document).on("click", ".logbook_refuse_signature_btn", function(event) {
		var info = $(this).attr("name").split("__");
		requestLogbookRefuseSignature(info[1], info[2], true);		
		return false;
	});
	
	$(document).on("click", ".logbook_thumbnail_image_item", function(event) {
		var info = $(this).parent().attr("id").split("__");
		var data = $(this).parent().attr("data-ad");
		
		if(data != null) {
			g_logbookIsPostedAd = true;
			data = data.split("__");
			requestAdClickContent("Content_Logbook", data[0], data[1]);
		}
		else
			g_logbookIsPostedAd = false;	
			
		requestLogbookDetail(info[1], info[2], false);
		
		return false;
	});
	
	$(document).on("click", ".logbook_thumbnail_image_detail_item", function(event) {		
		setLogbookGalleryView();
		return false;
	});
		
	$(document).on("click", ".logbook_thumbnail_image_remove_btn", function(event) {
		var id = $(this).attr("id");
		var info = id.split("__");
		
		setAlertMessage(lang["confirm_remove_file"]);
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: lang["yes"],
	            	click: function() {
	            		$(this).dialog("close");					
	            		requestLogbookRemoveFile(info[1], info[2], info[3]);
	            		$("#" + id).parent().parent().remove();	
	            	}
	            },	            
	            {
	            	text: lang["no"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]
		});
		
		return false;
	});
	
	$(document).on("click", "#datepicker_logbook", function(event) {
		$(this).datepicker({
			showOn: "button",
			buttonImage: "./main/images/btn_date_pick.png",
			buttonImageOnly: true,
			changeMonth: true,
			changeYear: true,
			showMonthAfterYear: true,
			yearRange: "c-60:c+10",
			monthNames: [lang["month_jan"], lang["month_feb"], lang["month_mar"], lang["month_apr"], lang["month_may"], lang["month_jun"], lang["month_jul"], lang["month_aug"], lang["month_sep"], lang["month_oct"], lang["month_nov"], lang["month_dec"]],
			monthNamesShort: ['1','2','3','4','5','6','7','8','9','10','11','12'],
			dayNames: [lang["day_sun"], lang["day_mon"], lang["day_tue"], lang["day_wed"], lang["day_thur"], lang["day_fri"], lang["day_sat"]],
			dayNamesShort: [lang["day_sun"], lang["day_mon"], lang["day_tue"], lang["day_wed"], lang["day_thur"], lang["day_fri"], lang["day_sat"]],
			dayNamesMin: [lang["day_sun"], lang["day_mon"], lang["day_tue"], lang["day_wed"], lang["day_thur"], lang["day_fri"], lang["day_sat"]],
			dateFormat: lang["date_format"]
		});
		
		return false;
	});
				
	$(document).on("click", "#upload_logbook", function(event) {
		$(this).uploadify({
			method: "post",
		    fileTypeDesc: lang["image_files"],
		    fileTypeExts: "*.jpg; *.jpeg; *.gif",
		    buttonImage: "main/images/btn_picture.png",
		    width: 25,
		    height: 25,
			swf: "_base/uploadify/uploadify.swf",
	        uploader: "/logbook/upload_files.dive",
	        multi: true,
	        auto: false,
	        onQueueComplete: function(queueData) {
	        	hidePopup();
	        	exportLogbookAd();
	        	refreshLogbookListView(g_logbookCurrentPageNumber);	        	
	        }
		});
		
		return false;
	});
	
	$(document).on("click", "input[name=signeeName]", function(event) {
		initBuddySelectView("radio", "logbook_buddy_select_ok_btn");
		return false;
	});	
	
	$(document).on("click", "input[name=findSigneeBtn]", function(event) {		
		initBuddySelectView("radio", "logbook_buddy_select_ok_btn");
		return false;
	});
	
	$(document).on("click", ".logbook_buddy_select_ok_btn", function(event) {
		setLogbookSigneeInfo();
		hidePopup();	
		return false;
	});
	
	$(document).on("click", ".logbook_write_cancel_btn", function(event) {
		if(g_mainSelectedMenu == "news")
			refreshNewsListView();
		else
			handleLogbookMenu("logbook_list_view");
		
		return false;
	});	
	
	$(document).on("click", "#log_book .category a", function(event) {		
		$(this).addClass('active');
		$(this).parent().siblings().find('a').removeClass('active');	
		
		changeLogbookCategory($(this).attr("href"));
				
		return false;
	});
	
	$(document).on("click", ".sticker .category a", function(event) {
		$(this).parents(".list").find("a").removeClass("active");
		$(this).addClass('active');
		
		return false;
	});
	
	$(document).on("blur", ".dive_time", function(event) {
		autoTimeOut();
		return false;
	});
	
	$(document).on("blur", ".time_in_hour", function(event) {
		autoTimeOut();		
		return false;
	});
	
	$(document).on("blur", ".time_in_minute", function(event) {
		autoTimeOut();
		return false;
	});	
	
	$(document).on("click", ".dive_setting button", function(event) {
		$(this).siblings().fadeToggle();		
		return false;
	});	
	
	$(document).on("click", "button[name=logbook_sticker_view_btn]", function(event) {
		if(g_logbookStickerView.length == 0)
			requestLogbookSticker();
		else
			setLogbookStickerView(null);
			
		return false;
	});
	
	$(document).on("click", "input[name=logbook_add_sticker_btn]", function(event) {
		if(g_logbookSelectedSticker.length == 0)
			showMessage(lang["sticker_select"]);
		else
			setLogbookAddSticker(g_logbookSelectedSticker);
		
		return false;
	});
	
	$(document).on("click", "input[name=logbook_cancel_add_sticker_btn]", function(event) {
		hidePopup();
		return false;
	});
	
	$(document).on("click", "input[name=logbook_video_link_btn]", function(event) {
		setLogbookVideoLinkView();
		return false;
	});
	
	$(document).on("click", ".logbook_help_import_btn", function(event) {
		window.open("./main/pages/import.jsp", "_blank");
		return false;
	});
	
	$(document).on("change", "select[name=distanceUnit]", function(event) {
		$(".distanceUnit").text(getDistanceUnit($(this).val()));
		return false;
	});
	
	$(document).on("change", "select[name=temperatureUnit]", function(event) {
		$(".temperatureUnit").text(getTemperatureUnit($(this).val()));
		return false;
	});
	
	$(document).on("change", "select[name=weightUnit]", function(event) {
		$(".weightUnit").text(getWeightUnit($(this).val()));
		return false;
	});
	
	$(document).on("change", "select[name=pressureUnit]", function(event) {
		$(".pressureUnit").text(getPressureUnit($(this).val()));
		return false;
	});
	
	$(document).on("change", "#log_month select", function(event) {
		requestLogbookStatisticsPerMonth(g_userId, $(this).val());
		return false;
	});
	
	$(document).on("click", ".sticker .contain .emoticon a", function(event) {
		g_logbookSelectedSticker = getHref($(this).attr("href"));
		return false;
	});
	
	$(document).on("click", ".emoticon button", function(event) {
		var id = $(this).attr("name");		
		
		setAlertMessage(lang["confirm_remove_sticker"]);
		$("#alert").dialog({
			resizable: false,
			height: 140,
			modal: true,
			buttons: [
	            {
	            	text: lang["yes"],
	            	click: function() {
	            		$(this).dialog("close");
	            		$("#" + id).remove();
	            	}
	            },	            
	            {
	            	text: lang["no"],
	            	click: function() {
	            		$(this).dialog("close");
	            	}
	            }
	        ]
		});
		
		return false;
	});
	
	$(document).on("click", "#graph_view_canvas", function(event) {
		if(location.href.indexOf("index.jsp") > -1)
			setLogbookProfileDetailView();
		
		return false;
	});
	
	$(document).on("click", "button[name=logbook_graph_detail_btn]", function(event) {
		setLogbookProfileDetailView();
		return false;
	});
	
	$(document).on("click", "#logbook_bulk_import_list .cancel_btn", function(event) {
		var files = document.getElementById("log_files").files;
		var index = $(this).attr("data-index");
		var item = $("#logbook_import_file_" + index);
		
		item.attr("data-canceled", "true");
		item.hide();
		
		g_logbookImportTotalCount--;
		
		if(getSelectedImportFileCount(files) == 0)
			initLogbookBulkImportChooseFilesView();
		
		return false;
	});
	
	$(document).on("mousemove", "#graph_view_canvas", function(event) {
		setLogbookGraphValue(event);		
		return false;
	});
	
	$(document).on("touchmove", "#graph_view_canvas", function(event) {
		setLogbookGraphValue(event);
		return false;
	});
	
	$(document).on("mousemove", "#graph_detail_canvas", function(event) {
		setLogbookGraphDetailValue(event);		
		return false;
	});
	
	$(document).on("touchmove", "#graph_detail_canvas", function(event) {
		setLogbookGraphDetailValue(event);
		return false;
	});
	
	$(document).on('click','#logbook_photo_view .showcase_image .prev button',function(){
		$('.showcase_image .cycle-slideshow').cycle('prev');
		var idx = $('.showcase_image .cycle-slideshow').find('.cycle-slide-active').index();
		$('.showcase_thumb .list a').removeClass('active');
		$('.showcase_thumb .cycle-slide-active a:eq('+((idx-1)%15)+')').addClass('active');
		if(idx%15==0){
			$('.showcase_thumb .cycle-slideshow').cycle('prev');
			$('.showcase_thumb .cycle-slide-active a:eq('+((idx-1)%15)+')').addClass('active');
		}
		return false;
	});
	
	$(document).on('click','#logbook_photo_view .showcase_image .next button',function(){
		$('.showcase_image .cycle-slideshow').cycle('next');
		var idx = $('.showcase_image .cycle-slideshow').find('.cycle-slide-active').index();
		$('.showcase_thumb .cycle-slideshow a').removeClass('active');
		$('.showcase_thumb .cycle-slide-active a:eq('+((idx-1)%15)+')').addClass('active');
		if(idx%16==0){
			$('.showcase_thumb .cycle-slideshow').cycle('next');
			$('.showcase_thumb .cycle-slide-active a:eq('+((idx-1)%15)+')').addClass('active');
		}
		return false;
	});
	
	$(document).on('click','#logbook_photo_view .showcase_thumb .prev',function(){		
		$('.showcase_thumb .cycle-slideshow').cycle('prev');
		$('.showcase_thumb .cycle-slide-active a:eq(0)').addClass('active');
		$('.showcase_thumb .cycle-slide-active a:eq(0)').siblings().removeClass('active');
		var current = ($('.showcase_thumb').find('.cycle-slide-active').index()-1)*15;
		$('.showcase_image .cycle-slideshow').cycle('goto',current);
		return false;
	});
	
	$(document).on('click','#logbook_photo_view .showcase_thumb .next',function(){		
		$('.showcase_thumb .cycle-slideshow').cycle('next');
		$('.showcase_thumb .cycle-slide-active a:eq(0)').addClass('active');
		$('.showcase_thumb .cycle-slide-active a:eq(0)').siblings().removeClass('active');

		var current = ($('.showcase_thumb').find('.cycle-slide-active').index()-1)*15;
		$('.showcase_image .cycle-slideshow').cycle('goto',current);
		return false;
	});
	
	$(document).on('click','#logbook_photo_view .showcase_thumb .list a',function(){
		var idx = $(this).attr("href");
		$('.showcase_image .cycle-slideshow').cycle('goto',idx);
		$(this).siblings().removeClass('active');
		$(this).addClass('active');
		return false;
	});	
});


////////////////////////////////////////////////////////////////////////////////////////////
/// Request Data
////////////////////////////////////////////////////////////////////////////////////////////

function requestLogbookList(userId, condition, pageNumber) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/list.dive?userId=" + userId + "&pageNumber=" + pageNumber + "&pageSize=" + g_logbookPageSize,
  		data: condition,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookListView(resultMap.data, pageNumber);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookCount(userId, condition) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/count.dive?userId=" + userId,
  		data: condition,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) 				
  				setLogbookPageView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookSignedList(userId, condition, pageNumber) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/signed_list.dive?userId=" + userId + "&pageNumber=" + pageNumber + "&pageSize=" + g_logbookPageSize,
  		data: condition,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookListView(resultMap.data, pageNumber);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookStatisticsOverview(userId) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/statistics.dive?userId=" + userId,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookStatisticsOverviewView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookStatisticsPerYear(userId, year) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/statistics_year.dive?userId=" + userId,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookStatisticsPerYearView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookStatisticsMinimumYear(userId) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/statistics_minimum_year.dive?userId=" + userId,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				initLogbookStatisticsPerMonthView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookStatisticsPerMonth(userId, year) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/statistics_month.dive?userId=" + userId + "&year=" + year,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookStatisticsPerMonthView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookSignedCount(userId, condition) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/signed_count.dive?userId=" + userId,
  		data: condition,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) 				
  				setLogbookPageView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookBestList(count) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/best_list.dive?count=" + count,  		
  		dataType: "json", 
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				setLogbookBestListView(resultMap.data);
  		}			
	});
}

function requestLogbookDetail(userId, logbookNumber, usedModify) {
	hidePopup();
	
	$.ajax({
  		type: "GET",
  		url: "/logbook/detail.dive?deviceType=Web&userId=" + userId + "&logbookNumber=" + logbookNumber,
  		data: "referer=" + document.referrer,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {
  				if(usedModify)
  					initLogbookModifyView(resultMap.data);
  				else
  					setLogbookDetailView(resultMap.data);
  			}  	  				
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookExport(target, userId, logbookNumber) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/detail.dive?deviceType=Web&userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				exportLogbook(target, resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookPdfConversion(userId, logbookNumber) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/pdfConversion.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				alert("succeeded");
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookWriteForm() {
	$.ajax({
  		type: "GET",
  		url: "/logbook/template.dive",
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))  				
  				setLogbookEditView("new", resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookTemplateForImport() {
	$.ajax({
  		type: "GET",
  		url: "/logbook/template.dive",
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {
  				resultMap.data.diveNumber--;
  				readLogFiles("Bulk", 0, resultMap.data);
  			}	
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookCopiedWriteForm(userId, logbookNumber) {
	hidePopup();

	$.ajax({
  		type: "GET",
  		url: "/logbook/copied_template.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) 
  				setLogbookEditView('copy', resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookWrite(request_data) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/write.dive",
  		data: request_data,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {  				
  				g_logbookCurrentPageNumber = 1;
  				uploadLogbook("write", g_myId, resultMap.data, g_logbookCurrentPageNumber);
  			}
  			else
  				showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookImport(type, index, format, data, command) {
	var url = "/logbook/import.dive?type=";
	var form = null;	
	
	if(type == "Bulk") {
		url += type;
		
		command.diveNumber++;
		command.deviceType = "Web",
		command.format = format;
		command.data = data;
		
		data = command;
	}
	else {
		if(type == "new") {
			url += 'New';
			form = $("form[name=logbook_write_form]");
			
			generateLogbookStickers();
			
			if($("#datepicker_logbook").datepicker("option", "dateFormat") != "yy-mm-dd")
				$("#datepicker_logbook").datepicker("option", "dateFormat", "yy-mm-dd");
		}			
		else {		
			if(type == "modify")			
				form = $("form[name=logbook_modify_form]");
			else
				form = $("form[name=logbook_modify_restricted_form]");
			
			var info = form.attr("action").split("__");
			url += "Modify&userId=" + info[1] + "&logbookNumber=" + info[2];
		}
		
		form.find("input[name=deviceType]").val("Web");
		form.find("input[name=format]").val(format);
		form.find("input[name=data]").val(data);
		
		data = form.serialize();
	}
	
	$.ajax({
  		type: "POST",
  		url: url,
  		data: data,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(type == "Bulk")
  				setLogbookBulkImportResult(index, isSucceeded(resultMap), resultMap.result.message, command);
  			else {  				
  				if(isSucceeded(resultMap))  				
  	  				refreshLogbookListView(g_logbookCurrentPageNumber);
  	  			else
  	  				showResultMessage(resultMap);	
  			}
  		},
  		error: function(xhr, status, msg) { 
  			if(type == "Bulk")
  				setLogbookBulkImportResult(index, false, msg, command);
  			else
  				showErrorMessage(xhr, status, msg); 
		},
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookModify(userId, logbookNumber, request_data) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/modify.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		data: request_data,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				uploadLogbook("modify", userId, logbookNumber, g_logbookCurrentPageNumber);
  			else
  				showResultMessage(resultMap);         
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookModifyRestricted(userId, logbookNumber, request_data) {
	$.ajax({
  		type: "POST",
  		url: "/logbook/modify_restricted.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		data: request_data,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				uploadLogbook("modify_restricted", userId, logbookNumber, g_logbookCurrentPageNumber);
  			else
  				showResultMessage(resultMap);         
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookDelete(userId, logbookNumber) {
	hidePopup();
	
	$.ajax({
  		type: "GET",
  		url: "/logbook/delete.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				refreshLogbookListView(g_logbookCurrentPageNumber);
  			else
  				showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookSort() {
	$.ajax({
  		type: "GET",
  		url: "/logbook/sort.dive?userId=" + g_myId,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {
  				showMessage(lang["completed_sort"]);
  				if(g_logbookSelectedMenu != "logbook_bulk_import_view")
  					refreshLogbookListView(g_logbookCurrentPageNumber);
  			}  				
  			else
  				showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookPutSignature(userId, logbookNumber, viewCheck) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/put_signature.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) { 
  				if(viewCheck) {
  					setAlertMessage(lang["confirm_put_signature"]);
  					$("#alert").dialog({
  						resizable: false,
  						height: 140,
  						modal: true,
  						buttons: [
				            {
				            	text: lang["yes"],
				            	click: function() {
				            		$(this).dialog("close");
				            		g_logbookIsPostedAd = false;
				            		requestLogbookDetail(userId, logbookNumber, false);
				            	}
				            },	            
				            {
				            	text: lang["no"],
				            	click: function() {
				            		$(this).dialog("close");
				            	}
				            }
				        ]
  					});		
  				}
  				else {
  					g_logbookIsPostedAd = false;
  					requestLogbookDetail(userId, logbookNumber, false);
  				}  					
  			}
  			else
  				showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookRefuseSignature(userId, logbookNumber, viewCheck) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/refuse_signature.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {  				
  				if(viewCheck) {
  					g_logbookIsPostedAd = false;
  					requestLogbookDetail(userId, logbookNumber, false);
  				}  					
  				else
  					showMessage(lang["refused_signature"]);	
  			}
  			else
  				showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookLike(userId, logbookNumber) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/like.dive?userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				setLogbookLikeCount(userId, logbookNumber, resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookRemoveFile(userId, logbookNumber, fileNumber) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/remove_file.dive?userId=" + userId + "&logbookNumber=" + logbookNumber + "&fileNumber=" + fileNumber,
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

function requestLogbookSticker() {
	$.ajax({
  		type: "GET",
  		url: "/logbook/stickers.dive",
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap))
  				setLogbookStickerView(resultMap.data);
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookAd(userId, logbookNumber, post) {
	var url = '';
	
	if(post)
		url = "/ad/add_content_item.dive";
	else
		url = "/ad/remove_content_item_by_item.dive";
	
	$.ajax({
  		type: "GET",
  		url: url + "?type=Content_Logbook&userId=" + userId + "&itemNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false,
  		success: function(resultMap) {
  			if(isSucceeded(resultMap)) {
  				if(post)
  					showMessage(lang["ad_content_posted_item"]);
  				else
  					showMessage(lang["ad_content_stop_item"]);	
  			}  				 
  			else
	            showResultMessage(resultMap);
  		},
  		error: function(xhr, status, msg) { showErrorMessage(xhr, status, msg); },
  		beforeSend: function() { showProgress(); },
  		complete: function() { hideProgress(); }
	});
}

function requestLogbookShare(target, userId, logbookNumber) {
	$.ajax({
  		type: "GET",
  		url: "/logbook/share.dive?target=" + target + "&userId=" + userId + "&logbookNumber=" + logbookNumber,
  		dataType: "json",
  		cache: false
	});
}


////////////////////////////////////////////////////////////////////////////////////////////
/// Check Validation
////////////////////////////////////////////////////////////////////////////////////////////

function checkLogbook(form) {
	// diveNumber
	var diveNumber = form.find("input[name=diveNumber]").val();
	
	if(diveNumber.length == 0) {
		showMessage(lang["dive_number_input"]); 
		return false;		
	}
	
	var expDiveNumber = /^[0-9]{1,5}$/;
	if(diveNumber.length < 0 || !expDiveNumber.test(diveNumber)) {
		showMessage(lang["dive_number_integer"]); 
		return false;
	}
	
	// diveDate
	var diveDate = form.find("input[name=diveDate]").val();
	
	if(diveDate.length == 0) {
		showMessage(lang["dive_date_input"]);  
		return false;		
	}	
	
	// location
	var location = form.find("input[name=location]").val();

	if(location.length == 0) {
		showMessage(lang["location_input"]);  
		return false;		
	}
	
	if(location.length > 100) {
		showMessage(lang["location_length"]); 
		return false;	
	}
	
	// point
	var point = form.find("input[name=point]").val();

	if(point.length > 0 && point.length > 100) {
		showMessage(lang["point_length"]);  
		return false;
	}
	
	// surfaceIntervalHour
	var surfaceIntervalHour = form.find("input[name=surfaceIntervalHour]").val();
	
	if(surfaceIntervalHour.length > 0) {
		var expSurfaceIntervalHour = /^[0-9]{1,2}$/;
		if(!expSurfaceIntervalHour.test(surfaceIntervalHour)) {
			showMessage(lang["si_hour_integer"]);  
			return false;
		}
	}
	
	// surfaceIntervalMinute
	var surfaceIntervalMinute = form.find("input[name=surfaceIntervalMinute]").val();
	
	if(surfaceIntervalMinute.length > 0) {
		var expSurfaceIntervalMinute = /^[0-9]{1,2}$/;
		if(!expSurfaceIntervalMinute.test(surfaceIntervalMinute) || surfaceIntervalMinute < 0 || surfaceIntervalMinute > 59) {
			showMessage(lang["si_minute_integer"]);  
			return false;
		}
	}	
	
	// maxDepth
	var maxDepth = form.find("input[name=maxDepth]").val();
	
	if(maxDepth.length > 0) {
		var expMaxDepth = /^([0-9]*)(\.?)([0-9]*)$/;
		if(!expMaxDepth.test(maxDepth)) {
			showMessage(lang["max_depth_number"]);   
			return false; 
		}
	}
	
	// averageDepth
	var averageDepth = form.find("input[name=averageDepth]").val();
	
	if(averageDepth.length > 0) {
		var expAverageDepth = /^([0-9]*)(\.?)([0-9]*)$/;
		if(!expAverageDepth.test(averageDepth)) {
			showMessage(lang["average_depth_number"]);  
			return false;
		}
	}
	
	// diveTime
	var diveTime = form.find("input[name=diveTime]").val();
	
	if(diveTime.length > 0) {
		var expDiveTime = /^[0-9]{1,}$/;
		if(!expDiveTime.test(diveTime)) {
			showMessage(lang["dive_time_integer"]);  
			return false;
		}
	}
	
	// safetyStopTime
	var safetyStopTime = form.find("input[name=safetyStopTime]").val();
	
	if(safetyStopTime.length > 0) {
		var expSafetyStopTime = /^[0-9]{1,}$/;
		if(!expSafetyStopTime.test(safetyStopTime)) {
			showMessage(lang["safety_stop_time_integer"]);   
			return false;
		}
	}
	
	// timeInHour
	var timeInHour = form.find("input[name=timeInHour]").val();
	
	if(timeInHour.length > 0) {
		var expTimeInHour = /^[0-9]{1,2}$/;
		if(!expTimeInHour.test(timeInHour) || timeInHour < 0 || timeInHour > 23) {
			showMessage(lang["time_in_hour_integer"]);   
			return false;
		}
	}
	
	// timeInMinute
	var timeInMinute = form.find("input[name=timeInMinute]").val();
	
	if(timeInMinute.length > 0) {
		var expTimeInMinute = /^[0-9]{1,2}$/;
		if(!expTimeInMinute.test(timeInMinute) || timeInMinute < 0 || timeInMinute > 59) {
			showMessage(lang["time_in_minute_integer"]);   
			return false;
		}
	}
	
	// timeOutHour
	var timeOutHour = form.find("input[name=timeOutHour]").val();
	
	if(timeOutHour.length > 0) {
		var expTimeOutHour = /^[0-9]{1,2}$/;
		if(!expTimeOutHour.test(timeOutHour) || timeOutHour < 0 || timeOutHour > 23) {
			showMessage(lang["time_out_hour_integer"]);   
			return false;
		}
	}
	
	// timeOutMinute
	var timeOutMinute = form.find("input[name=timeOutMinute]").val();
	
	if(timeOutMinute.length > 0) {
		var expTimeOutMinute = /^[0-9]{1,2}$/;
		if(!expTimeOutMinute.test(timeOutMinute) || timeOutMinute < 0 || timeOutMinute > 59) {
			showMessage(lang["time_out_minute_integer"]);   
			return false;
		}
	}
	
	// startTankPressure
	var startTankPressure = form.find("input[name=startTankPressure]").val();
	
	if(startTankPressure.length > 0) {
		var expstartTankPressure = /^[0-9]{1,}$/;
		if(!expstartTankPressure.test(startTankPressure)) {
			showMessage(lang["start_tank_pressure_integer"]);    
			return false;
		}
	}
	
	// endTankPressure
	var endTankPressure = form.find("input[name=endTankPressure]").val();
	
	if(endTankPressure.length > 0) {
		var expendTankPressure = /^[0-9]{1,}$/;
		if(!expendTankPressure.test(endTankPressure)) {
			showMessage(lang["end_tank_pressure_integer"]);   
			return false;
		}
	}
	
	// enrichedAirBlend
	var enrichedAirBlend = form.find("input[name=enrichedAirBlend]").val();
	
	if(enrichedAirBlend.length > 0) {
		var expEnrichedAirBlend = /^([0-9]*)(\.?)([0-9]*)$/;
		if(!expEnrichedAirBlend.test(enrichedAirBlend)) {
			showMessage(lang["enriched_air_blend_number"]);    
			return false;
		}
	}
	
	// weight
	var weight = form.find("input[name=weight]").val();
	
	if(weight.length > 0) {
		var expWeight = /^[0-9]{1,}$/;
		if(!expWeight.test(weight) || weight < 0) {
			showMessage(lang["weight_integer"]);    
			return false; 
		}
	}
	
	// visibility
	var visibility = form.find("input[name=visibility]").val();
	
	if(visibility.length > 0) {
		var expVisibility = /^[0-9]{1,}$/;
		if(!expVisibility.test(visibility) || visibility < 0) {
			showMessage(lang["visibility_integer"]);   
			return false;
		}
	}
	
	
	// maxTemperature
	var maxTemperature = form.find("input[name=maxTemperature]").val();
	
	if(maxTemperature.length > 0) {
		var expmaxTemperature = /^(-?)([0-9]*)(\.?)([0-9]*)$/;
		if(!expmaxTemperature.test(maxTemperature)) {
			showMessage(lang["max_temperature_number"]);   
			return false;
		}
	}
	
	// minTemperature
	var minTemperature = form.find("input[name=minTemperature]").val();
	
	if(minTemperature.length > 0) {
		var expminTemperature = /^(-?)([0-9]*)(\.?)([0-9]*)$/;
		if(!expminTemperature.test(minTemperature)) {
			showMessage(lang["min_temperature_number"]);     
			return false;
		}
	}
	
	// activity
	var activity = form.find("input[name=activity]").val();
		
	if(activity.length > 0 && activity.length > 100) {
		showMessage(lang["activity_length"]);    
		return false;
	}
	
	// note
	var note = form.find("textarea[name=note]").val();
	
	if(note.length > 0 && note.length > 3000) {
		showMessage(lang["note_length"]);    
		return false;
	}
	
	return true;
}

function checkLogbookRestricted(form) {
	// note
	var note = form.find("textarea[name=note]").val();
	
	if(note.length > 0 && note.length > 3000) {
		showMessage(lang["note_length"]);   
		return false;
	}
	
	return true;
}



