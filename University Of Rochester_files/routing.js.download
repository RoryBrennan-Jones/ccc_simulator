/*-----------------------------------------------------*\
    @GLOBAL variables
\*-----------------------------------------------------*/
    // [global] isTransitionPageNav : defaults to false, set to true when transitionPageNav() is called, set back to false in [accessibility.js] setPageName()
    // Set to true on page transition; setPageName() will update document.title when isTransitionPageNav is true (then set isTransitionPageNav=false)
    // Page title will not be altered for modals (primary-modal/secondary-modal) or Thickbox/iFrame windows (only on page load or page transition)
	var isTransitionPageNav = false;

/* Function called to navigate within the Account Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionAccountNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "account", true, actionObject);
	}

/* Function called to navigate within the Admin Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionAdminNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "admin", true, actionObject);
	}

/* Function called to navigate within the App Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionAppNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "app", true, actionObject);
	}

/* Function called to navigate within the Fair Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionFairNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "fair", true, actionObject);
	}

/* Function called to navigate within the Groups Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionGroupsNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "groups", true, actionObject);
	}

/* Function called to navigate within the Groups Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionEventsNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "events", true, actionObject);
	}

/* Function called to navigate within the Home Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionHomeNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
		transitionPageNav(strURL, "home", true, actionObject);
	}

/* Function called to navigate within the Manage Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionManageNav(strURL) {
		var actionObject = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : null;
	  	transitionPageNav(strURL, "manage", true, actionObject);
	}

/* Internal function called to navigate handle AJAX calls to reload LeftMenu & Content.
	* @param strURL     	  		: The URL to go to
	* @param strType     	  		: The menu type
	* @param boolPushState    		: Possible additional parameter to specify whether to push a history state or not.
	* @param btnElement       		: Within actionObject - The button element (for collapsing current menu feature)
*/
	function transitionPageNav(strURL, strType, boolPushState, actionObject) {
        isTransitionPageNav = true;

		//Redirect when coming from Page where Left Menu does NOT exist.
		if ($("#sidebar").length == 0) { 
			window.location.href = strURL;
			return;
		}

		//Action Object Properties
		var btnElement 			  = actionObject !== null ? actionObject.btnElement : null;
		var appId			  	  = actionObject !== null ? actionObject.appId : null;
		var alerts			  	  = actionObject !== null ? actionObject.alerts : null;

		if (!isEmpty(btnElement)) {
			if (btnElement.hasAttribute('aria-expanded')) {
				if ($(btnElement).attr('aria-expanded') === 'true') {
					// if this sidebar menu is expanded - we'll collapse it (and all others are assumed to be collapsed) - set aria-expanded to false for all expandable sidebar menus
					$('#sidebar a[aria-expanded="true"]').attr('aria-expanded', 'false');
				} else if ($(btnElement).attr('aria-expanded') === 'false') {
					// if this sidebar menu is collapsed - we're expanding it - set aria-expanded to false for all then set to true just for the immediately expanded sub-menu
					$('#sidebar a[aria-expanded="true"]').attr('aria-expanded', 'false');
					$(btnElement).attr('aria-expanded', 'true');
				}
			}
		}

		let boolReloadSidebar = false;

		//First Condition is Regular Navigation - Second Condition is from PopState (Back & Forth buttons).
		if ((history.state !== null && history.state.type != strType) || (typeof currentHistoryType !== "undefined" && currentHistoryType !== null && currentHistoryType.type != strType)) {
			transitionLeftMenu(strType, appId);
			transitionTopMenu(strType);

			// Need to call again to fix bug that sometimes happens.
			initLeftMenuCollapse();

			// Set boolean to trigger correct event.
			boolReloadSidebar = true
		}
		transitionContent(strURL, boolReloadSidebar, alerts);
		
		//Collapse Menu
		if (btnElement && hasActiveChild(btnElement)) {
			$(window).trigger("hashchange", true);
			return;
		}

		//URL Change		
		if (boolPushState) { history.pushState({type: strType, appId: appId}, "", strURL); }
		$(window).trigger("hashchange");
		
		return false;
	}

/* Function called to navigate within the Account Section.
	* @param strURL     	  : The URL to go to
*/
	function transitionLeftMenu(strType, appId) {
		appId = appId || "";
		// Add Skeleton
		const strSkeletonMenu = "<ul class='metisMenu nav' id='side-menu'>" + 
									"<li style='margin-top: 15px; padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
									"<li style='padding: 5px 15px;'><div class='skeleton' style='border-radius: 49.9%;min-height: 20px; width: 20px;margin-left: 10px;display: inline-block;'></div><div class='skeleton' style='min-height: 20px;display: inline-block;width: 145px;margin-left: 25px;'></div></li>" + 
							  	"</ul>"
		$("#sidebar-menu").html(strSkeletonMenu);

		// Left Menu AJAX
		$.ajax({
			method: "GET",
			url: "/sidebar",
			data: { m: strType, id: appId },
			success: function (html) {
				$("#sidebar").replaceWith(html);

				//Re-Run Scroll + Collapse Init from /static/js/menu.js
				initMetisMenu();
				initSlimscrollMenu();
				initLeftMenuCollapse();
				initEnlarge();

				const transitionSidebarEvent = new CustomEvent('transitionSidebar');
				document.dispatchEvent(transitionSidebarEvent);	
			}, 
			error: function (rep) {
				if (rep.status == 403 || rep.status == 404 || rep.status == 500) {
					//console.log(rep.status);
				}
			}
		});
	}


/* Function called to activate the correct Top Menu Container.
	* @param strType     	  : The menu type
*/
	function transitionTopMenu(strType) {
		$("li[id^='header__btn-cont--']").removeClass("active");
		if (strType != "manage") {
			$("#header__group-icon").attr("data-original-title", "Groups");
			$("#header__group-text").html("Groups");
		}

		$("#header__btn-cont--" + strType).addClass("active");
	}

var routingXHR;
/* Function called to load content for any Navigation Type.
	* @param strURL     	  : The URL to go to
	* @param boolReloadSidebar : Boolean indicating if the sidebar was reloaded.
	** Uses the routingXHR global variable **
*/
	function transitionContent(strURL, boolReloadSidebar, alerts) {

        const topMainContentElementHTML = "<div style='position: relative;'><span id='span-top-of-main-content--0' class='content__top-element' tabindex='0' aria-label='Top of Main Content.' style='display: none;'>Top of Main Content</span></div>";
        if (routingXHR && routingXHR.readyState != 4) {
			routingXHR.abort();
		}
		$("#page-cont").html('<div style="margin-top:300px;"></div>' + writeLoading("Loading page..."));
        
        const url = new URL((strURL.indexOf("http") !== 0 ? window.location.origin : '') + strURL);
        url.searchParams.append('ax', '1');
        if(alerts){
        	if(alerts.error){
				url.searchParams.append('cg-error', alerts.error);
			}
		}
        strURL = url.href;

		routingXHR = $.ajax({
			url: strURL,
			success: function (html) {
				$("#page-cont").html(topMainContentElementHTML + html);
				if(document.getElementById('generic')) {
					document.getElementById('generic').style.display = 'none';
				}

				// Trigger window.load event manually
				dispatchEvent(new Event('load'));

				if (!boolReloadSidebar) {
					const transitionContentEvent = new CustomEvent('transitionContent');
					document.dispatchEvent(transitionContentEvent);	
				}
			}, 
			error: function (rep) {
				if (rep.status == 403) {
					//location.reload()
					$("#page-cont").html(rep.responseText);
				}
				else if (rep.status == 404 || rep.status == 500) {
					//location.reload()
				}
			}
		});
	}


var currentHistoryType;

/* Initiated here and updated in the override pushSate */
history.previousUrl = new URL(location.href);

/* 
	** Override the pushState native methode to provide a previousUrl variable.
*/
var oldPushState = history.pushState;
history.pushState = function pushState(a, b, c) {
	oldPushState.apply(history, [a, b, c]);
	history.previousUrl = new URL(location.href);
	currentHistoryType = a;
}

/* Listener to manage previous and forward buttons */
window.onpopstate = function(e) {
	if (history.previousUrl != null && history.previousUrl.pathname == new URL(location.href).pathname 
	&& history.previousUrl.searchParams.get("show") == new URL(location.href).searchParams.get("show")
	&& history.previousUrl.searchParams.get("mode") == new URL(location.href).searchParams.get("mode")
	&& history.previousUrl.searchParams.get("view") == new URL(location.href).searchParams.get("view")) {
		if (typeof setFiltersFromURLParameters == 'function') { 
		  	setFiltersFromURLParameters(); 
			getDataFromWebService();
			history.previousUrl = new URL(location.href);
			return;
		}
	}

	var eStateType = (e.state === null) ? null : e.state.type;
	var actionsObject = {}
	if (e.state !== null && e.state.appId){
		actionsObject.appId = e.state.appId;
	}
	transitionPageNav(document.location.pathname + document.location.search, eStateType, false, actionsObject);
	currentHistoryType = { type: eStateType, appId: actionsObject.appId }

	
	history.previousUrl = new URL(location.href);
};