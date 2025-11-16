/*-----------------------------------------------------*\
    @GLOBAL variables
\*-----------------------------------------------------*/

    /* Global constant isAccessibilityJSDefined defaults to true (used by Website Builder to determine if we're in the editor or on a group website) */
    const isAccessibilityJSDefined = true;

    /* Global variable isAccessibilityJSVerbose
        * ALWAYS set this to FALSE before committing code (to be phased out).
        * Set to true for debug console output.
    */    
    var isAccessibilityJSVerbose = false;

    /* Global variable isShiftTabNavigationPending used to signal/flag SHIFT-TAB has been pressed
        * Used for back-navigation with the keyboard from (1) top of main-content back to left-navigation or (2) top of left-nav back to the top header
    */
    var isShiftTabNavigationPending = false;

    /* Global value cgTrapKeyboardTrapTabIndexReduction used when trapping keyboard functionality inside an iFrame or modal
        * We remove keyboard functionality (outside the iFrame/modal) by subtracting this value from the tabindex of all elements outside the iFrame/modal
        * We restore keyboard functionality (when the iFrame/modal is closed) by adding this value back to the tabindex of all other elements on the page
        * Works well with multiple levels of modals open (ex. secondary modal opened from primary modal)
    */
    const cgTrapKeyboardTrapTabIndexReduction = 1000;

    /* Global array numberNames : The names of the numbers 0-19
        * Used in getAriaNumberName(n)
    */
    var numberNames = ['zero', 'one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten', 'eleven', 'twelve', 'thirteen', 'fourteen', 'fifteen', 'sixteen', 'seventeen', 'eighteen', 'nineteen' ];

    /* Global array tensNames : The names of the "tens" component of a number (blank for numbers less than 20 as these don't have an associated "tens" name component)
        * Used in getAriaNumberName(n)
    */
    var tensNames = ['', '', 'twenty', 'thirty', 'fourty', 'fifty', 'sixty', 'seventy', 'eighty', 'ninety' ];

    /* Global variable setFocusOnTopMessagesEpoch used to prevent multiple calls to setFocusOnTopMessages()
        * We'll only handle the first of N calls within a 100 millisecond timespan
    */
    var setFocusOnTopMessagesEpoch = 0;

    /* Global variable isClick set to true when an action is a click and false when it comes from keyboard
    */
    var isClick = true;

    /* Global stack for tracking open/close of modals, dialogs, iFrames, etc.
        * ID of the modal/dialog/iFrame being opened is pushed onto dialogCallStack
        * stack is popped in reverse order when associated dialogs are closed
        * call stack is used to manage tabindex reduction for keyboard lock on the backsheet
        * when AJAX calls return data that is rendered behind the top (active) popup
        * NOTE: The same dialog ID can not be pushed twice in a row (you can't open a dialog on top of itself)
        * If the dialog referenced by containerId is not on the top of the call stack, it's likely we're (accidentally)
        * closing a modal, dialog or iFrame behind the top (active) dialog. In this case, we'll first attempt to properly close all open dialogs
        * on top of the dialog requesting closure.
        * FUTURE: a better solution in the future may be to properly handle ESCAPE and associated dialog close requests to either
        *         (i) always be caught by the top (active) dialog with stopPropogation OR
        *         (ii) indepdently (and properly) close all dialogs stacked on top of the dialog requesting closure - outside of this function
        * Until we choose a better solution, we'll continue to manage multi-dialog closures within popDialogCallStack() below.
    */
    var dialogCallStack = [];

    /* Global counter to track consecutive mousedown events - if more than 3x mouse clicks we'll remove body.acc-keyboard-mode
        * 
    */
    var countConsecutiveMousedowns = 0;

    /* Global variable bypassFocusToMainContent set to true when opening an expandable sidebar menu. In keyboard mode,
        * focus should move to the first sub-menu item inside an expandable menu when the menu is opened.
    */
    var bypassFocusToMainContent = false;

/*-----------------------------------------------------*\
    @LISTENERS AND BINDINGS

    Some full document bindings and custom event listeners
    to take appropriate actions when users appear to be
    using the platform via keyboard or with a screen reader.
\*-----------------------------------------------------*/

    document.addEventListener("DOMContentLoaded", function(event) {

        document.addEventListener('mousedown', function() {
            countConsecutiveMousedowns++;
            if (countConsecutiveMousedowns >= 3) {
                if (document.body.classList.contains('acc-keyboard-mode')) {
                    document.body.classList.remove('acc-keyboard-mode');
                }
            }
        });

        document.addEventListener('click', function() { 
            isClick = true;
        });

        document.addEventListener('keydown', function(e) { 
            setTimeout(function () {
                isClick = false; // all keydown's flag isClick=false (including ENTER)
                if (isShiftTabKey(e) || isTabKey(e) || isArrowKey(e)) {
                    countConsecutiveMousedowns = 0;
                    if (!document.body.classList.contains('acc-keyboard-mode')) {
                        document.body.classList.add('acc-keyboard-mode');
                        setupSlickAccKeyboardMode();
                    }
                }
            }, 3); //  wait for 'click' to be handled (will set isClick=true) - 3 ms works well in testing
        });
    });

    /* Custom event listeners */

    document.addEventListener('transitionSidebar', function(){
        if (isAccKeyboardMode()) {
            bypassFocusToMainContent = true;
            setupStrCheckboxForLists();
            setTimeout(function() {
                if (isSidebarCollapsed()) {
                    navigateToActiveMenuItem();
                }
                setFocusToSidebar();
            }, 1);
        }
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupA11yDragDropKeyboardSupport();
    });
    document.addEventListener('transitionContent', function(){
        if (isAccKeyboardMode()) {
            setupStrCheckboxForLists();
            setFocusToContent();
        }
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupA11yDragDropKeyboardSupport();
    });
    document.addEventListener('openModal', function(e){
        setModalDialogTitle(e.detail.modalId);
        if (isAccKeyboardMode()) {
            setFocusToModal(e.detail.modalId);
        }
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupA11yDragDropKeyboardSupport();
    });
    document.addEventListener('transitionModal', function(e){
        setModalDialogTitle(e.detail.modalId);
        if (isAccKeyboardMode()) {
            setFocusToModal(e.detail.modalId);
        }
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupA11yDragDropKeyboardSupport();
    });
    document.addEventListener('openDialog', function(e){
        setModalDialogTitle(e.detail.modalId);
        if (isAccKeyboardMode()) {
            setFocusToModal(e.detail.dialogId);
        }
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupA11yDragDropKeyboardSupport();
    });
    document.addEventListener('ajaxLoadMore', function(e){
        let parentContainerId = e.detail.parentContainerId;
        if (isEmpty(parentContainerId)) { parentContainerId = null; }
        setupAjaxAccessibilityForLoadMore(parentContainerId)
        setupStrCheckboxForLists();
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
    });

/*-----------------------------------------------------*\
    @DOCUMENT READY LAUNCH and AJAX call returns

    Calls to accessibility functions on page ready and
    AJAX call returns.
\*-----------------------------------------------------*/

    document.addEventListener("DOMContentLoaded", function() {
        // KEYBOARD CALLS
        handleKeyboardForTopbarDropdowns();

	    // Restore keyboard navigation (remove keyboard trap from backsheet when the primary or secondary modal closes)
	    $("#primary-modal").on('hide.bs.modal', function () { terminatePopupAccessibility('primary-modal', 'modal'); });
	    $("#secondary-modal").on('hide.bs.modal', function () { terminatePopupAccessibility('secondary-modal', 'modal'); });
        setAriaCurrentState();
        processAriaLive();
        setupStrCheckboxForLists();
        addAriaLabelToExpandableSidebarLinks();
        updateUserTagsBackgroundColors();
        updateBadgeBackgroundColors();
        setupTracksAndChecklistsAccessibility();
        setupA11yDragDropKeyboardSupport();
    });

    /* Function called to reset accessibility function on AJAX calls. */
    function setupAjaxAccessibility() {
        addShiftTabListeners();
        addKeyboardClickListeners();
        setPageName();

        setupSkipToLeftNavigation();
        addAriaLabelsToOrderingSelectOptions();

        setupTooltipAccessibility();
        hideDecorativeImagesAndIconsFromScreenReader();
        setAriaCurrentState();
        addAriaLabelToExpandableSidebarLinks(); 
        setupTracksAndChecklistsAccessibility();
        setupA11yDragDropKeyboardSupport();
    }

    /* Function called to hook in required accessibility for "Load More" AJAX calls
        * OR standard AJAX calls that take longer to load and may not return before
        * the user opens a dialog, modal or iFrame (i.e. new data loading on the backsheet
        * after keyboard lock)
        * 
    */
    function setupAjaxAccessibilityForLoadMore(parentContainerId) {
        addKeyboardClickListeners();

        setupTooltipAccessibility();
        hideDecorativeImagesAndIconsFromScreenReader();
        verifyKeyboardLock(parentContainerId); // sync tabindex in regions behind modal/dialog on late load (or live/active areas)
        setupA11yDragDropKeyboardSupport();
    }

/*-----------------------------------------------------*\
    @CONTRAST RATIO

    Calculate color contrast ratio between text and background (or between two colors)

\*-----------------------------------------------------*/

    /*
     * Function that converts a color value into an {r, g, b, a} property set where r,g,b are in the range 0-255 and a is between 0 and 1
     * @param color : A color value such as #112233 or rgba(255, 100, 33, 1) or hsla(360, 0.5, 0.3, 1) or hsla(360, 50%, 30%, 1)
     * 
     * Properties included in the return value:
     * 
     * r = red
     * g = green
     * b = blue
     * a = alpha
     * isRGBA = true
     * 
     * The values r, g and b are in the range 0-255 and a is in the range 0-1.
     * Also includes property isRGBA set to true.
     */
    function getRGBA(color) {
        if (isEmpty(color)) { console.error('color is empty'); return; }
        if (color.isRGBA === true) { return color; }
        if (color.isHSLA === true) { return convertHSLAToRGBA(color); }
        if (!isString(color)) { console.error('color is not valid'); return; }
        if (color.startsWith('hsl')) { return convertHSLAToRGBA(color); }
        
        let r = 0;
        let g = 0;
        let b = 0;
        let a = 1.0;

        if (color.startsWith('rgb')) {
            color = color.replace('rgb(', '').replace('rgba(', '').replace(')', '');
            color = color.split(',');
            r = parseInt(color[0]);
            g = parseInt(color[1]);
            b = parseInt(color[2]);
            if (color.length === 4) {
                a = parseFloat(color[3]);
            }
        } else {
            if (color.startsWith('#')) {
                color = color.substring(1);
            }
            if (color.length<6) {
                console.error('> color value is invalid for color=' + color);
                return;
            }

            r = '' + color[0] + color[1];
            g = '' + color[2] + color[3]
            b = '' + color[4] + color[5]
            a = 'ff';
            if (color.length==8) {
                a = '' + color[6] + color[7]
            }

            r = parseInt(r, 16);
            g = parseInt(g, 16);
            b = parseInt(b, 16);
            a = parseInt(a, 16);

            a /= 255;
        }

        return { r: r, g: g, b: b, a: a, isRGBA : true };
    }

    /*
     * Function that converts an RGBA color value to an HSLA property set.
     * @param rgba : rgba color value in string or RGBA format
     * 
     * Properties in return value:
     * 
     * h = Hue
     * s = Saturation
     * l = Lightness
     * a = Alpha
     * isHSLA = true
     * 
     */
    function convertRGBAToHSLA(color) {
        let rgba = getRGBA(color);
        if (isEmpty(rgba) || rgba.isRGBA !== true) {
            console.error('rgba is invalid');
            return;
        }

        /*
         * CONVERT rgb values to [0,1]
         */
        let r = rgba.r / 255;
        let g = rgba.g / 255;
        let b = rgba.b / 255;
        let a = rgba.a;

        let cMin = Math.min(r, g, b);
        let cMax = Math.max(r, g, b);
        let delta = cMax - cMin;

        let h = 0;
        let s = 0;
        let l = 0;

        /*
         * CALCULATE Hue
         */
        if (delta === 0) {
            // h = 0
        } else if (cMax === r) {
            h = ((g - b) / delta) % 6;
        } else if (cMax === g) {
            h = (b - r) / delta + 2;
        } else if (cMax === b) {
            h = (r - g) / delta + 4;
        }
        h = Math.round(h * 60);
        if (h < 0) {
            h += 360;
        }

        /*
         * CALCULATE Lightness
         */
        l = (cMax + cMin) / 2;

        /*
         * CALCULATE Saturation
         */
        if (delta === 0) {
            s = 0;
        } else {
            s = delta / (1 - Math.abs(2 * l - 1));
        }

        return { h: h, s: s, l: l, a: a, isHSLA: true };
    }

    /* Function that converts a color value into an {h, s, l, a} property set.
     * @param color : A color value such as #112233 or rgba(255, 100, 33, 1) or hsla(360, 0.5, 0.3, 1) or hsla(360, 50%, 30%, 1)
     *
     * Properties in return value:
     * 
     * h = Hue
     * s = Saturation
     * l = Lightness
     * a = Alpha
     * isHSLA = true
     * 
     * Where h is in the range [0,360] and {s, l, a} are in the range [0, 1]
     * Also includes property isHSLA set to true.
     */
    function getHSLA(color) {
        if (isEmpty(color)) { console.error('color is empty'); return; }
        if (color.isHSLA === true) { return color; }
        if (color.isRGBA === true) { return convertRGBAToHSLA(color); }
        if (!isString(color)) { console.error('color is not valid'); return; }

        if (color.startsWith('rgb') || color.startsWith('#')) {
            return convertRGBAToHSLA(color);
        } else if (color.startsWith('hsl') && color.includes('(') && color.includes(')')) {
            let sx = color.indexOf('(') + 1;
            let ex = color.indexOf(')');
            let parts = color.substring(sx, ex);
            parts = parts.split(',');
            let h = Math.floor(parseFloat(parts[0]));
            let s = 0;
            if (parts[1].includes('%')) {
                s = 0.01 * parseFloat(parts[1].replace('%', ''));
            } else {
                s = parseFloat(parts[1]);
            }
            let l = 0;
            if (parts[2].includes('%')) {
                l = 0.01 * parseFloat(parts[2].replace('%', ''));
            } else {
                l = parseFloat(parts[2]);
            }
            let a = 1;
            if (parts.length > 3) {
                a = parseFloat(parts[3]);
            }
            return { h: h, s: s, l: l, a: a, isHSLA: true };
        }
    }

    /* Function that converts an HSLA color to RGBA format.
     * @param color : a color value
     *
     * Properties included in the return value:
     * 
     * r = red
     * g = green
     * b = blue
     * a = alpha
     * isRGBA = true
     * 
     * The values r, g and b are in the range 0-255 and a is in the range 0-1.
     * Also includes property isRGBA set to true.
     */
    function convertHSLAToRGBA(color) {
        let hsla = getHSLA(color);
        if (isEmpty(hsla) || hsla.isHSLA !== true) {
            console.error('> hsla is invalid');
            return;
        }

        let C = (1 - Math.abs(2 * hsla.l - 1)) * hsla.s;
        let X = C * (1 - Math.abs((hsla.h / 60) % 2 - 1));
        let m = hsla.l - C / 2;

        let r0 = 0;
        let g0 = 0;
        let b0 = 0;
        if (hsla.h < 60) {
            r0 = C;
            g0 = X;
            b0 = 0;
        } else if (hsla.h < 120) {
            r0 = X;
            g0 = C;
            b0 = 0;
        } else if (hsla.h < 180) {
            r0 = 0;
            g0 = C;
            b0 = X;
        } else if (hsla.h < 240) {
            r0 = 0;
            g0 = X;
            b0 = C;
        } else if (hsla.h < 300) {
            r0 = X;
            g0 = 0;
            b0 = C;
        } else if (hsla.h < 360) {
            r0 = C;
            g0 = 0;
            b0 = X;
        }
        let r = (r0 + m) * 255;
        let g = (g0 + m) * 255;
        let b = (b0 + m) * 255;

        if (r > 255) { r = 255; }
        if (g > 255) { g = 255; }
        if (b > 255) { b = 255; }

        let rgba = {r: Math.floor(r + 0.5), g: Math.floor(g + 0.5), b: Math.floor(b + 0.5), a: hsla.a, isRGBA: true};
        return rgba;
    }

    /*
     * Function that calculates the relative Luminance for a given RGBA value normalized to values between 0 and 1
     * @param color : a color value in hex format, rgb(), rgba(), hsla() etc.
     */
    function getRelativeLuminance(color) {
        color = getRGBA(color);
        if (isEmpty(color)) { console.error('color is empty'); return; }
        if (color.isRGBA !== true) { console.error('color is invalid'); return; }

        let r = color.r / 255;
        let g = color.g / 255;
        let b = color.b / 255;

        let R = 0;
        if (r < 0.03928) {
            R = r / 12.92;
        } else {
            R = Math.pow((r + 0.055) / 1.055, 2.4);
        }

        let G = 0;
        if (g < 0.03928) {
            G = g / 12.92;
        } else {
            G = Math.pow((g + 0.055) / 1.055, 2.4);
        }

        let B = 0;
        if (b < 0.03928) {
            B = b / 12.92;
        } else {
            B = Math.pow((b + 0.055) / 1.055, 2.4);
        }

        let L = 0.2126 * R + 0.7152 * G + 0.0722 * B;

        return L;
    }

    /* Function that returns the visible RGB value represented by an RGBA foreground color (with alpha between 0 and 1) on an RGB background (with alpha 1.0) */
    function getVisibleRGB(rgbaForeground, rgbaBackground) {
        rgbaForeground = getRGBA(rgbaForeground);
        rgbaBackground = getRGBA(rgbaBackground);
        if (isEmpty(rgbaForeground)) { console.error('> ERROR rgbaForeground is empty'); return; }
        if (isEmpty(rgbaBackground)) { console.error('> ERROR rgbaBackground is empty'); return; }
        if (rgbaBackground.a < 1) {
            rgbaBackground = getVisibleRGB(rgbaBackground, '#ffffff');
        }

        let oneMinusA = 1.0 - rgbaForeground.a;
        let r = rgbaForeground.r * rgbaForeground.a + rgbaBackground.r * oneMinusA;
        let g = rgbaForeground.g * rgbaForeground.a + rgbaBackground.g * oneMinusA;
        let b = rgbaForeground.b * rgbaForeground.a + rgbaBackground.b * oneMinusA;

        r = Math.min(255, Math.max(0, Math.round(r)));
        g = Math.min(255, Math.max(0, Math.round(g)));
        b = Math.min(255, Math.max(0, Math.round(b)));

        return getRGBA('rgb(' + r + ',' + g + ',' + b + ')');
    }

    /*
     * Function that returns the contrast ratio between two color value. This implementation matches the WAVE Plugin implementation as of March 2022.
     * @param color1 : foreground color (usually text or icon)
     * @param color2 : background color
     * 
     * Returns a value between 1 and 21. For example, the standard WCAG Level AA color contrast ratio minimum is 4.5 for regular text and 3.0 for large text.
     */
    function getColorContrastRatio(color1, color2) {
        color1 = getVisibleRGB(color1, color2);
        var rlum1 = getRelativeLuminance(color1);
        if (isEmpty(rlum1)) {
            console.error('rlum1 is empty');
            return;
        }
        var rlum2 = getRelativeLuminance(color2);
        if (isEmpty(rlum2)) {
            console.error('rlum2 is empty');
            return;
        }
        let max = Math.max(rlum1, rlum2);
        let min = Math.min(rlum1, rlum2);
        return (max + 0.05) / (min + 0.05);
    }

    /* Function that returns a text string usable as a color value in element.style such as background-color
     * @param color : a color value
     *
     * Returns an rgba(r, g, b, a) or hsla(h, s, l, a) color value depending on the value passed in color.
     */
    function getHtmlColorText(color, useHexFormat) {
        if (color.isHSLA === true && useHexFormat !== true) {
            let hText = '' + (Math.floor(100 * color.h + 0.5) / 100);
            let sText = '' + (Math.floor(10000 * color.s + 0.5) / 100) + '%';
            let lText = '' + (Math.floor(10000 * color.l + 0.5) / 100) + '%';
            let aText = '' + (Math.floor(1000 * color.a + 0.5) / 1000);
            let htmlColorText = 'hsla(' + hText + ', ' + sText + ', ' + lText + ', ' + aText + ')';
            return htmlColorText;
        }
        color = getRGBA(color);
        if (isEmpty(color) || color.isRGBA !== true) { console.error('color is invalid'); return; }

        if (useHexFormat === true) {
            let rText = '' + color.r.toString(16);
            let gText = '' + color.g.toString(16);
            let bText = '' + color.b.toString(16);
            let aText = '';
            if (rText.length < 2) { rText = '0' + rText; }
            if (gText.length < 2) { gText = '0' + gText; }
            if (bText.length < 2) { bText = '0' + bText; }
            if (color.a < 1) {
                let a255 = Math.floor(color.a * 255);
                if (a255 > 255) {
                    a255 = 255;
                } else if (a255 < 0) {
                    a255 = 0;
                }
                aText = a255.toString(16);
                if (aText.length < 2) {
                    aText = '0' + aText;
                }
            }
            return '#' + rText + gText + bText + aText;
        }

        let aText = '' + (Math.floor(1000 * color.a + 0.5) / 1000);
        return 'rgba(' + color.r + ', ' + color.g + ', ' + color.b + ', ' + aText + ')';
    }

/*-----------------------------------------------------*\
    @KEYBOARD

    Setup and manage keyboard events 
    Functions to check what key is pressed

\*-----------------------------------------------------*/

    /* Function that returns true if the current user appears to be using keyboard navigation and/or a screen reader
    */
    function isAccKeyboardMode() {
        return $("body").hasClass("acc-keyboard-mode");
    }

    // DROPDOWN
    /* Function called to handle keyboard navigation for dropdown in the topbar. */
    function handleKeyboardForTopbarDropdowns() {
        
        // $(this) is either $topbarButton or $topbarDropdownContainer
        // Need setTimeout because document.activeElement becomes body before focus is set
        function onDropdownFocusOut(e) {
            if (!isClick) {
                $topbarFocusOutElement = $(this);
                setTimeout(function () {
                    if ($topbarFocusOutElement.prop("tagName") == "BUTTON") {
                        let topbarDropdownContainer = $topbarFocusOutElement.siblings("ul")[0];
                        if (!(topbarDropdownContainer == document.activeElement || topbarDropdownContainer.contains(document.activeElement))) {
                            $topbarFocusOutElement.parent().removeClass("open");
                        }
                    }
                    else if ($topbarFocusOutElement.prop("tagName") == "UL") {
                        topbarButton = $topbarFocusOutElement.siblings("button")[0];
                        if (!(topbarButton == document.activeElement || $topbarFocusOutElement[0].contains(document.activeElement))) {
                            $topbarFocusOutElement.parent().removeClass("open");
                        }                  
                    }
                }, 50, $topbarFocusOutElement);
            }
        }
        $("#topbar li.dropdown").each(function() {
            let $topbarButton = $(this).find("> button");
            let $topbarDropdownContainer = $(this).find("> ul");

            if ($topbarButton.length > 0) {
                let elem = $topbarButton[0];
                $topbarButton[0].removeEventListener('focusout', onDropdownFocusOut);
                $topbarButton[0].addEventListener('focusout', onDropdownFocusOut);
            }
            if ($topbarDropdownContainer.length > 0) {
                $topbarDropdownContainer[0].removeEventListener('focusout', onDropdownFocusOut);
                $topbarDropdownContainer[0].addEventListener('focusout', onDropdownFocusOut);
            }
        });
    }

    /* Function called to check if a keydown event is Shift + Tabs
        * SHIFT-TAB, shift-tab and backwards navigation are all the same (relates to keyboard user going "backwards" through the focusable DOM elements)
        * @param e : Keydown Event
    */
    function isShiftTabKey(e) {
        return e.which === 9 && e.shiftKey && !e.altKey && !e.ctrlKey;
    }
    function isArrowKey(e) {
        return (e.which === 37 || e.which === 38 || e.which === 39 || e.which === 40) &&
                !e.shiftKey && !e.altKey && !e.ctrlKey;
    }
    function isTabKey(e) {
        return e.which === 9 && !e.shiftKey && !e.altKey && !e.ctrlKey;
    }

    /* Function that returns the active top navigation element (Home, Event, Admin...). 
        * Default is Hamburger button
    */ 
    function getTopbarActiveElement() {
        var $topNavigationActiveElements = $('#topbar li.active a');
        if ($topNavigationActiveElements.length > 0) {
            return $topNavigationActiveElements[0];
        }
        return $('#button-menu-mobile')[0];
    }

    /* Function that returns the active left navigation element (Sub-Menu Or Menu). 
        * Default is getTopbarActiveElement. 
    */ 
    function getSidebarActiveElement() {
        var $leftNavActiveSubMenuItemElements = $('#sidebar li.active li.active a.active');
        if ($leftNavActiveSubMenuItemElements.length > 0) {
            return $leftNavActiveSubMenuItemElements[0];
        }
        var $leftNavActiveMenuItemElements = $('#sidebar li.active a.active');
        if ($leftNavActiveMenuItemElements.length > 0) {
            return $leftNavActiveMenuItemElements[0];
        }
        return getTopbarActiveElement();
    }

    /* Function that returns all sidebar active menu items. 
    */ 
    function getSidebarActiveElements() {
        return $('#sidebar li.active > a.active');
    }

    /* Function that returns the main content top element */
    function getMainContentTopElement() {
        var $mainContentTopElements = $("#span-top-of-main-content--0");
        if ($mainContentTopElements.length > 0) {
            return $mainContentTopElements[0];
        }
    }

    /* Function that handles shift tab event. Set global variable to true to trigger action on focusout.
        * @param e : Keydown Event
    */
    function handleShiftTabKeydown(e) {
        if (isShiftTabNavigationPending) { return; } 
        
        if (isShiftTabKey(e)) {
            isShiftTabNavigationPending = true;
        }
    }

    /* Function that handles shift tab event on main content. Set focus on left navigation active element (or top navigation active element if left navigation is not present).
        * @param e                  : Keydown Event 
        * @param focusTargetElement : Element to receive focus
    */
    function handleShiftTabFocusout(e, focusTargetElement) {
        if (!isShiftTabNavigationPending) { return; }

        isShiftTabNavigationPending = false;

        if (isEmpty(focusTargetElement)) {
            return;
        }
        focusTargetElement.focus();
    }

    /* Function that handles shift tab event on main content. Set focus on left navigation active element (or top navigation active element if left navigation is not present).
        * @param e : Keydown Event 
    */
    function handleMainContentShiftTabFocusout(e) {
        if (!isShiftTabNavigationPending) { return; }
        if (isSidebarCollapsed() && isAccKeyboardMode()) {
            navigateToActiveMenuItem();
        }
        handleShiftTabFocusout(e, getSidebarActiveElement());
    }

    /* Function that handles shift tab event on left navigation. Set focus on top navigation active element.
        * @param e : Keydown Event 
    */
    function handleLeftNavigationShiftTabFocusout(e) {
        handleShiftTabFocusout(e, getTopbarActiveElement());
    }

    /* Function that adds event listeners for keydown/focusout from the top of main content and top of left navigation
        * Catch shift-tab backwards navigation out of a region so we can forced focus to a parent menu button or menu item
    */
    function addShiftTabListeners() {
        var $leftNavTopElements = $("#sidebar #sidebar-menu ul li a");
        if ($leftNavTopElements.length > 0) {
            var leftNavTopElement = $leftNavTopElements[0]; // the top one (if multiple are active for some reason)
            leftNavTopElement.removeEventListener('keydown', handleShiftTabKeydown);
            leftNavTopElement.removeEventListener('focusout', handleLeftNavigationShiftTabFocusout);
            leftNavTopElement.addEventListener('keydown', handleShiftTabKeydown);
            leftNavTopElement.addEventListener('focusout', handleLeftNavigationShiftTabFocusout);
        }
        var mainContentTopElement = getMainContentTopElement();
        if (!isEmpty(mainContentTopElement)) {
            mainContentTopElement.removeEventListener('keydown', handleShiftTabKeydown);
            mainContentTopElement.removeEventListener('focusout', handleMainContentShiftTabFocusout);
            mainContentTopElement.addEventListener('keydown', handleShiftTabKeydown);
            mainContentTopElement.addEventListener('focusout', handleMainContentShiftTabFocusout);
        }
    }

    /* Function that adds keyup listener to an element to capture ENTER/SPACE and call the element's .click()
        * Note the element must already support being clicked by the mouse (.click() must already do something)
        * @param element : The element that requires ENTER/SPACE assistance to call the .click()
    */
    function addKeyboardClickListener(element) {
        function handleKeyboardClick(e) {
            if (e.which === 13 || e.which === 32) {
                e.preventDefault();
                e.stopPropagation();

                // prevent multiple accidental clicks on "stuck" enter key (for example - enter pressed on modal 'x' close button should not press enter on this element)
                if (checkLastKeyclick(element) < 500) { return; }

                element.click();
                flagLastKeyclick(element);
            }
        }
        // add tabindex="0" to ensure the element can take keyboard focus (but do not change the tabindex if already defined)
        if (!isTabIndexSpecified(element)) {
            $(element).attr('tabindex', '0');
        }
        // Remove existing keyup listener to avoid multiple events firing
        $(element).off('keyup.data-keyclick');
        $(element).on('keyup.data-keyclick', handleKeyboardClick);
    }

    /* Function that assigns the epoch millis of the last click to an element that was clicked
        * used together with checkLastKeyclick() below to determine the milliseconds since the
        * last time the associated element was clicked (in order to prevent fast user actions
        * from accidentally causing a recurring click loop, ex. press enter to close a modal
        * and the enter is also read by the associated backpage element taking focus on modal
        * close - resulting in the modal being opened once again when you're actually trying to close it!)
    */
    function flagLastKeyclick(element) {
        if ($(element).attr('data-keyclick') !== 'true') {return;}
        let epochNow = (new Date()).getTime();
        $(element).attr('data-epochLastKeyclick', '' + epochNow);
    }

    /* Function that checks the milliseconds since the associated element was clicked
        * to prevent accidental double clicks or accidental re-focus clicks when a modal is closed
        * called from function addKeyboardClickListener(element) -> inner function handleKeyboardClick(e)
    */
    function checkLastKeyclick(element) {
        if ($(element).attr('data-keyclick') !== 'true') {return;}
        let epochNow = (new Date()).getTime();
        let epochLastClick = $(element).attr('data-epochLastKeyclick');
        if (isEmpty(epochLastClick)) {
            epochLastClick = 0;
        } else {
            epochLastClick = parseInt(epochLastClick);
        }
        return epochNow - epochLastClick;
    }

    /* Function called to associate keyboard ENTER/SPACE with elements that define the attribute data-keyclick=true
        * only use for elements that do not already click when keyboard ENTER/SPACE is pressed
        * adds event listeners to click when ENTER/SPACE are pressed
    */
    function addKeyboardClickListeners() {
        var $clickableElements = $('[data-keyclick="true"]');
        for (var i = 0; i < $clickableElements.length; i++) {
            addKeyboardClickListener($clickableElements[i]);
        }
    }

    /* Function that hides or shows the Skip to Navigation link in the top skip-links (show if the left navigation menu is present, even if hidden)
        * Hide the "Skip to Navigation" link if the left navigation menu is not present (it may be present but hidden in high-zoom or on smaller browser windows)
    */
    function setupSkipToLeftNavigation() {
        var skipToNavigationElement = document.getElementById('span-skip-to-left-navigation');
        if (isEmpty(skipToNavigationElement)) { return; }

        if ($('ul#side-menu li.menu-title:first').length > 0) {
            // DISPLAY Skip to Navigation (left navigation menu found)
            skipToNavigationElement.style.display = '';
        } else {
            // HIDE Skip to Navigation
            skipToNavigationElement.style.display = 'none';
        }
    }

    /* Function that defines breadcrumbs for keyboard users (activate navigation menu items are flagged with attribute aria-current=true).
        * The active topbar navigation items (and sidebar navigation items if present) are assigned attribute aria-current="true"
        * FUTURE: expand to include main-content tab bars (main content navigation regions)
    */
    function setAriaCurrentState() {
        $('*[aria-current]').removeAttr('aria-current');
        $(getTopbarActiveElement()).attr('aria-current', 'true');
        let sidebarActiveElements = getSidebarActiveElements();
        $(sidebarActiveElements).attr('aria-current', 'true');
        for (var i=0; i<sidebarActiveElements.length; i++) {
            let sidebarActiveElement = sidebarActiveElements[i];
            let closestUl = $(sidebarActiveElement).closest('ul');
            if (closestUl.length > 0) {
                if ($(closestUl).hasClass('nav-second-level')) {
                    $(closestUl).attr('aria-expanded', 'true');
                    $(closestUl).attr('data-a11y-sub-menu', 'true');
                } else {
                    $(sidebarActiveElement).attr('aria-expanded', 'true');
                    $(sidebarActiveElement).attr('data-a11y-menu-option', 'true');
                }
            }            
        }
    }

    /* Function that calls jQuery slideToggle() to expand or collapse a section and also updates the associated link's aria-expanded state
     * based on the state after slideToggle() is called
     * @param {HTMLElement} toggleLink - the link (or element) that is clicked to toggle the expandable section
     * @param {string} toggleSelector - unique CSS selector for the expandable section to toggle
     * @param {number} speed - milliseconds >= 1 to wait before toggling the section (passed to jQuery slideToggle())
    */
    function a11ySlideToggle(toggleLink, toggleSelector, speed) {
        if (!toggleLink) { console.error('toggleLink is empty'); return; }
        if (!toggleSelector) { console.error('toggleSelector is empty'); return; }
        if (!speed || !isNumber(speed)) { console.error('speed is empty or zero'); return; }

        let toggleElems = document.querySelectorAll(toggleSelector);
        if (toggleElems.length < 1) { console.error('toggleSelector target not found'); return; }
        if (toggleElems.length > 1) { console.error('toggleSelector target not unique'); return; }

        let toggleElem = toggleElems[0];
        $(toggleElems).slideToggle(speed, function() {
            let isExpanded = ($(toggleElem).css('display') === 'none') ? false : true;
            if (isExpanded) {
                $(toggleLink).attr('aria-expanded', 'true');
            } else {
                $(toggleLink).attr('aria-expanded', 'false');
            }
        });
    }

/*-----------------------------------------------------*\
 * @TABSTOPS
 * 
 * Find an element or a tree of elements for different scenarios
 * 
\*-----------------------------------------------------*/

    /* Function that returns true if an element will take keyboard focus natively (by default)
        * These elements typically do not require tabindex="0"
        * @param element : The element in question
    */
    function isNativeFocusElement(element) {
        const defaultFocusNodeNames = ['A', 'AREA', 'BUTTON', 'DETAILS', 'INPUT', 'SELECT', 'TEXTAREA'];
        return defaultFocusNodeNames.includes(element.nodeName);
    }

    /* Function that returns true if an element has specifically assigned a tabindex value (is the tabindex specified for the element)
        * Asking for .attr('tabindex') will always return an integer (0 for elements that have not specified a tabindex)
        * We want to know if the actual HTML code has defined a tabindex attribute, or if a tabindex attribute has been added via JavaScript
        * @param element : The element in question
    */
    function isTabIndexSpecified(element) {
        if (isEmpty(element.getAttributeNode) || isEmpty(element.getAttributeNode("tabIndex")) || isEmpty(element.getAttributeNode("tabIndex").specified)) { return false; }
        return element.getAttributeNode("tabIndex").specified;
    }

    /* Function that returns true if an element requires a tabindex reduction when a modal opens (in order to trap the keyboad focus inside the modal that just opened)
        * Native focus elements always require a tabindex reduction on keytrap, as well as any elements with a tabindex specified
        * @param element : The element in question
    */
    function isTabIndexReductionRequiredOnKeyboardTrap(element) {
        if (isNativeFocusElement(element)) {
            return true;
        }
        return isTabIndexSpecified(element);
    }

    /* Function that returns a valid jQuery CSS selector for the associated DOM element
        * @param element : The element in question (DOM element)
        * Returns a selector that should uniquely select the element using $(selector) where $(selector).length === 1 and $(selector)[0] is the element itself
    */
    function getElementSelector(element) {
        if (isEmpty(element) || $(element).length !== 1) { console.error('element is invalid'); return ''; }

        let tagName = $(element).prop('tagName');
        if (isEmpty(tagName)) { console.error('tagName is empty'); return ''; }

        tagName = tagName.toLowerCase();
        if (tagName === 'body') {
            return tagName;
        }

        if (!isEmpty(element.id)) {
            let selector = tagName + '#' + element.id;
            try {
                if ($(selector).length === 1) { return selector; }
            } catch(e) { /* do nothing - selector is not valid (this is ok) */ }
        }

        if (!isEmpty(element.classList) && element.classList.length > 0) {
            let selector = tagName + '.' + element.classList.toString().split(' ').join('.');
            selector = selector.split('active').join('');
            while (selector.includes('..')) {
                selector = selector.split('..').join('.');
            }
            if (selector.endsWith('.')) {
                selector = selector.substring(0, selector.length-1);
            }
            try {
                if ($(selector).length === 1) { return selector; }
            } catch(e) { /* do nothing - selector is not valid (this is ok) */ }
        }

        let href = $(element).attr('href');
        if (!isEmpty(href) && !href.trim().startsWith('javascript')) {
            let selector = tagName + '[href="' + href.split('"').join('""') + '"]';
            try {
                if ($(selector).length === 1) { return selector; }
            } catch(e) { /* do nothing - selector is not valid (this is ok) */ }
        }

        let onclick = $(element).attr('onclick'); 
        if (!isEmpty(onclick)) {
            let selector = tagName + '[onclick="' + onclick.split('"').join('""') + '"]';
            try {
                if ($(selector).length === 1) { return selector; }
            } catch(e) { /* do nothing - selector is not valid (this is ok) */ }
        }

        // temporarily tag the element with the following random class (we'll remove this before return)
        let tagClass = 'tag-';
        tagClass = tagClass + '-' + (1000000000 + Math.floor(Math.random() * 1000000000));
        tagClass = tagClass + '-' + (1000000000 + Math.floor(Math.random() * 1000000000));
        $(element).addClass(tagClass);

        let parent = $(element).parent()[0];
        let siblings = $(parent).children(tagName);
        if (siblings.length > 1) {
            for (var i=0; i<siblings.length; i++) {
                if (siblings[i].classList.toString().includes(tagClass)) {
                    $(element).removeClass(tagClass);
                    return getElementSelector(parent) + ' > ' + tagName + ':nth-child(' + (i+1) + ')';
                }
            }
        }
        $(element).removeClass(tagClass);
        return getElementSelector(parent) + ' > ' + tagName;
    }

/*-----------------------------------------------------*\
 * @POPUPS
 * 
 * Accessibility functions to handle focus / keyboard trapping for Dialogs, jPrompts, Modals and Thickbox iFrames. In addition:
 * - update/restore page title on modal open/close
 * - set initial focus to a specific element (inside popup)
 * - return focus to the appropriate element (behind popup) when the popup closes
 * 
\*-----------------------------------------------------*/

    /* Function that returns an array of all elements beneath a given element (including the element itself) recursively
        * Get an umbrella of elements (in an array)
        * @param element  : Current element under inspection  
        * @param elements : Array of all elements gathered recursively
    */
    function getAllElementsInTree(element, elements) {
        if (isEmpty(element) || isEmpty(elements)) { return; }

        elements[elements.length] = element;

        var childElements = element.children;
        for (var i = 0; i < childElements.length; i++) {
            getAllElementsInTree(childElements[i], elements);
        }
    }

    /* Function that returns an array of all elements beneath a given element (including the element itself) recursively, but excluding a sub-branch beneath the element with id=containerId
        * An umbrella of elements (in an array), but excluding a sub-umbrella. Typically the container is a modal/iFrame or equivalent.
        * @param element      : Current element under inspection  
        * @param elements     : Array of all elements gathered recursively
        * @param containerId  : String ID of the element to be excluded, including all child elements of this element
    */
    function getAllElementsInTreeExcludingContainer(element, elements, containerId) {
        if (isEmpty(containerId)) {return getAllElementsInTree(element, elements); }
        if (isEmpty(element) || isEmpty(elements) || element.id === containerId) { return; }

        elements[elements.length] = element;

        var childElements = element.children;
        for (var i = 0; i < childElements.length; i++) {
            getAllElementsInTreeExcludingContainer(childElements[i], elements, containerId);
        }
    }

    /* Function that returns all elements in the document except those in the container with id=containerId (and excluding the container element itself)
        * Used to get all elements requiring tabIndex reduction to trap keyboard navigation (keyboard to be locked inside the container referenced by containerId)
        * @param doc          : The document from which you want to retrive all elements (excluding the element referenced by containerId and it's child elements)
        * @param containerId  : String ID of the element to exclude, along with all child elements under this element (the dialog/jPropt/modal/thickbox to lock keyboard inside of)
    */
    function getAllElementsNotInsideContainer(doc, containerId) {
        if (isEmpty(doc)) { return; }

        var elements = [];
        getAllElementsInTreeExcludingContainer(doc.body, elements, containerId);
        return elements;
    }

    /* Function that reduces the tabindex for all clickable elements in the targetElements array (in order to trap keyboard functionality inside an iFrame/dialog/modal for example)
        * Used to remove keyboard functionality from a parent window or elements outside a dialog/modal while it's open (trap keyboard inside the dialog/modal)
        * References global cgTrapKeyboardTrapTabIndexReduction (defined above) to reduce the tabindex
        * @param targetElements : Array of elements (the elements to have keyboard functionality removed)
    */
    function _removeKeyboardFunctionality(targetElements) {
        for (var i = 0; i < targetElements.length; i++) {
            var element = targetElements[i];
            if (isTabIndexReductionRequiredOnKeyboardTrap(element)) {
                element.tabIndex = element.tabIndex - cgTrapKeyboardTrapTabIndexReduction;
            }
        }
    }

    /* Function that restores the tabindex for all clickable elements in the targetElements array (in order to restore keyboard functionality when a dialog/modal/iFrame closes)
        * Used to restore keyboard functionality to a parent window or elements outside a dialog/modal when it closes
        * References global cgTrapKeyboardTrapTabIndexReduction (defined above) to restore the tabindex
        * @param targetElements : Array of elements (the elements to have keyboard functionality restored)
        * @DELETE
    */
    function _restoreKeyboardFunctionality(targetElements) {
        for (var i = 0; i < targetElements.length; i++) {
            var element = targetElements[i];
            if (isTabIndexReductionRequiredOnKeyboardTrap(element)) {
                var newTabIndex = element.tabIndex + cgTrapKeyboardTrapTabIndexReduction;
                // NEW elements might have been defined on the page AFTER we reduced the tabindex. These will have an unusually high newTabIndex value and should not be altered.
                if (newTabIndex < 100) {
                    // element is OK to have tabindex restored (restored value is in reasonable range)
                    element.tabIndex = newTabIndex;
                }
            }
        }
    }

    /* Function called when a modal, dialog or iFrame (Thickbox) is opened to push the ID of the associated dialog onto the global dialogCallStack
        * @param containerId : The ID of the modal, dialog or iFrame that is being opened, ex. primary-modal, secondary-modal
    */
    function pushDialogCallStack(containerId) {
        if (dialogCallStack[dialogCallStack.length-1] === containerId) {
            return;
        }
        dialogCallStack.push(containerId);
    }

    /* Function called when a modal, dialog or iFrame (Thickbox) is closed. The associated ID of the closed dialog is popped off the dialogCallStack
        * @param containerId : The ID of the modal, dialog or iFrame that is being closed, ex. primary-modal, secondary-modal
    */
    function popDialogCallStack(containerId) {
        if (dialogCallStack.length < 1) { return; }

        let count = 0; // prevent future updates causing too many loops here (should never be more than 5 dialogs stacked; really 2 or 3)
        let id = dialogCallStack[dialogCallStack.length-1];
        let isFound = (id === containerId);
        while (!isFound && dialogCallStack.length > 0 && count < 5) {
            if (id === 'secondary-modal' || id === 'primary-modal') {
                $('#' + id + ' button.close').click();
                id = (dialogCallStack.length > 0) ? dialogCallStack[dialogCallStack.length-1] : null;
            } else if (id !== 'TB_window') {
                closeDialog('#' + id);
                id = (dialogCallStack.length > 0) ? dialogCallStack[dialogCallStack.length-1] : null;
            } else {
                unlockKeyboardFromPopup(id);
                id = dialogCallStack.pop();
            }
            isFound = (id === containerId);
            count++;
        }
        if (isFound && dialogCallStack.length > 0) {
            dialogCallStack.pop();
        }
    }

    /* Function that locks keyboard on a popup 
        * @param popupId : The popup ID 
    */
    function lockKeyboardForPopup(popupId) { 
        var targetElements = getAllElementsNotInsideContainer(window.document, popupId);
        _removeKeyboardFunctionality(targetElements);
    }

    /* Function that unlocks keyboard from a popup 
        * @param popupId : The popup ID 
    */
    function unlockKeyboardFromPopup(popupId) {
        var targetElements = getAllElementsNotInsideContainer(window.document, popupId);
        _restoreKeyboardFunctionality(targetElements);
    }

    /* Function that initializes accessibility behavior for popup 
        * @param popupId : The popup ID 
        * @param pathToElementToFocusOnClose : Selector path to element to focus on close 
        *
    */
    function initPopupAccessibility(popupId, pathToElementToFocusOnClose) {
        pushDialogCallStack(popupId);
        lockKeyboardForPopup(popupId);
        $("#" + popupId).attr("data-pathfocusclose", pathToElementToFocusOnClose);
    }

    /* Function that terminates accessibility behavior for popup 
        * @param popupId : The popup ID 
    */
    function terminatePopupAccessibility(popupId) {
            if ($('#' + popupId).length < 1) {
                return;
            }
            $('#' + popupId).removeAttr('aria-label'); // clear dialog accessibility label
            unlockKeyboardFromPopup(popupId);
            popDialogCallStack(popupId);
            let pathToElementToFocusOnClose = $("#" + popupId).attr("data-pathfocusclose");
            setFocusToElementFromCssSelector(pathToElementToFocusOnClose);
            $("#" + popupId).attr("data-pathfocusclose", "");
    }
    
    /* Function that determines what modal, dialog or iFrame a particular element is in
        * Called from verifyKeyboardLock() to help determine the correct tabindex reduction for late loading elements in a locked region
        * The returned regionId represents the ID of the parent dialog to the element (based on the open dialogCallStack IDs registered)
        * @param element : a DOM element for which we want to determine the parent modal, dialog or iFrame (or if the element is behind all popups on the main body)
    */
    function getDialogRegionId(element) {
        for (var i=dialogCallStack.length-1; i>=0; i--) {
            let dialogId = dialogCallStack[i];
            if ($(element).parent('#' + dialogId).length > 0) {
                return dialogId;
            }
        }
        return 'body';
    }

    /* Function that verifies all elements in the associated parentContainerId are locked properly while an overlay/dialog is open
        * @param parentContainerId : the top element containing all elememts to be verified (ex. the ID of a parent DIV to which new elements were just added)
    */
    function verifyKeyboardLock(parentContainerId) {
        if (dialogCallStack.length === 0) { return; }

        // determine the tabindex reduction value for each sheet in the modal/dialog stack (each region)
        var tabindexReduction = 0;
        let tabindexReductionByRegionId = {};
        for (var i=dialogCallStack.length-1; i>=0; i--) {            
            let regionId = dialogCallStack[i]; // parent container for associated modal/dialog/iFrame
            tabindexReductionByRegionId[regionId] = tabindexReduction;
            tabindexReduction -= cgTrapKeyboardTrapTabIndexReduction;
        }
        tabindexReductionByRegionId['body'] = tabindexReduction;

        function fixKeyboardLock(element, currentRegionId) {
            if (!isEmpty(element.id) && !isEmpty(tabindexReductionByRegionId[element.id])) { currentRegionId = element.id; }

            const tabindexReduction = tabindexReductionByRegionId[currentRegionId];
            if (tabindexReduction === 0) { return; } 

            let hasTabindex = isTabIndexSpecified(element);
            if (hasTabindex || isFocusableElement(element)) {
                // some 3rd party libraries have used tabindex=200 but I have not seen higher than this
                // we'll use 300 as arbitrary an upper positive tabindex below (i.e. the max regular positive tabindex we may encounter)
                const tabindex = (hasTabindex) ? parseInt(element.getAttribute('tabindex')) : 0;
                if (tabindex >= (tabindexReduction + 300)) {
                    element.setAttribute('tabindex', tabindex + tabindexReduction);
                }
            }

            let childElements = element.children;
            for (var i = 0; i < childElements.length; i++) {
                fixKeyboardLock(childElements[i], currentRegionId);
            }
        }

        // DEFAULT to full dom scan (typically under 100 millis) unless parentContainerId provided
        // FUTURE expand on the below to work with selectors (and update 2+ containers based on selector instead of ID value) - requires loop
        let regionId = 'body';
        let parentContainerElement = window.document.body;
        if (!isEmpty(parentContainerId)) {
            // caller specified a parent container (via ID) to help restrict the work to be done and avoid conflicts
            let element = $('#' + parentContainerId)[0];
            if (!isEmpty(element)) {
                // this is element represented by #parentContainerId
                parentContainerElement = element; 

                // tabindex reduction will be based on the parent region (body, primary-modal, secondary-modal, TB_Window, etc.)
                regionId = getDialogRegionId(parentContainerElement);
            }
        }
        fixKeyboardLock(parentContainerElement, regionId);
    }

    // ----------
    // THICKBOX
    // ----------

    /* Function that waits for the thickbox dialog iframe to load then sets:
     *
     * - dialog role (on parent container)
     * - dialog accessibility label (set to dialog title/heading)
     * - dialog focus (to the first actionable element beneath the title bar)
     * 
     * NOTE we need to wait for dialog open/load work to complete before setting focus
     *      otherwise it will be set to the top of the iFrame (instead of the first actionable
     *      element/input/tab inside the iframe)
     */
    function setupThickboxDialogAccessibility() {
        // find thickbox iframe containers
        let tbContainerElems = $('#TB_window');
        if (tbContainerElems.length < 0) { return; }

        // tbContainerElem is the thickbox iframe container; we'll use this to set role=dialog and also set the dialog title accessibility label below
        let tbContainerElem = tbContainerElems[0];
        $(tbContainerElem).attr('role', 'dialog');

        // if keyboard mode was active in the parent window then enable it in the iFrame (focus visibility)
        if (isAccKeyboardMode()) {
            $('iframe#TB_iframeContent').contents().find('body:not(body.acc-keyboard-mode)').addClass('acc-keyboard-mode');
        }

        /* Function to find the modal/dialog title text inside the dialog based on priority ordered selectors */
        function findTitleText() {
            // future fixes can probably be performed by updating titleSelectors
            const titleSelectors = [ '.ttl', '.w_title', 'h1', 'h2' ];
            if ($('iframe#TB_iframeContent').length > 0) {
                if ($('iframe#TB_iframeContent').contents().length > 0) {
                    for (var i=0; i<titleSelectors.length; i++) {
                        if ($('iframe#TB_iframeContent').contents().find(titleSelectors[i]).length > 0) {
                            let titleElement = $('iframe#TB_iframeContent').contents().find(titleSelectors[i])[0];
                            return $(titleElement).text();
                        }
                    }
                    let titleText = $('iframe#TB_iframeContent').attr('title');
                    if (!isEmpty(titleText)) {
                        return titleText;
                    }
                }
            }
        }

        /* Function to find the modal/dialog optimal focus element inside the dialog based on priority ordered selectors */
        function findFocusContainer() {
            // future fixes can probably be performed by updating focusContainerSelectors
            const focusContainerSelectors = [ '.modal-body', '.w_content', '#page-cont', 'body' ];
            for (var i=0; i<focusContainerSelectors.length; i++) {
                if ($('iframe#TB_iframeContent').length > 0) {
                    if ($('iframe#TB_iframeContent').contents().length > 0) {
                        if ($('iframe#TB_iframeContent').contents().find(focusContainerSelectors[i]).length > 0) {
                            return $('iframe#TB_iframeContent').contents().find(focusContainerSelectors[i])[0];
                        }
                    }
                }
            }
        }

        let isTitleSet = false;
        let isFocusSet = false;

        let maxIntervals = 50;
        let intervalId = setInterval(function() {
            if (maxIntervals-- < 0) {
                setFocusThickboxIframe();
                clearInterval(intervalId);
                return;
            }
            if ($('iframe#TB_iframeContent').contents().length > 0) {
                if (!isTitleSet) {
                    let tbIFrameTitleText = findTitleText();
                    if (!isEmpty(tbIFrameTitleText)) {
                        $(tbContainerElem).attr('aria-label', getSafeAriaLabelTextTrim(tbIFrameTitleText));
                        isTitleSet = true;
                    }
                }
                if (!isFocusSet) {
                    let focusContainer = findFocusContainer();
                    if (!isEmpty(focusContainer)) {
                        let focusElement = findFirstActionableElementInsideContainer(focusContainer);
                        if (!isEmpty(focusElement)) {
                            setTimeout(function() {
                                // wait for browser to finish processing iFrame and contents before setting focus
                                // 111 milliseconds provides a smooth experience (but not too soon)
                                focusElement.focus();
                            }, 150);
                            isFocusSet = true; // set this immediately so we can clear the interval
                        }
                    }
                }
                if (isTitleSet && isFocusSet) {
                    setFocusThickboxIframe();
                    clearInterval(intervalId);
                }
            }
        }, 100);
    }

    /* Function that locks keyboard functionality inside a Thickbox iFrame window when it opens (remove keyboard functionality in the parent window of the Thickbox iFrame)
    */
    function lockKeyboardInsideThickbox() {
        pushDialogCallStack('TB_window');
        var targetElements = getAllElementsNotInsideContainer(window.parent.document, 'TB_iframeContent');
        _removeKeyboardFunctionality(targetElements);
        setTimeout(function() {
            // we've just removed keyboard functionality behind the iframe and the iframe is loading
            // 333 milliseconds provides smooth experience (but call too soon and it won't work as expected)
            setupThickboxDialogAccessibility();
        }, 333);
    }

    /* Function that restores keyboard functionaity to the parent window of a Thickbox iFrame (when the Thikbox iFrame closes)
    */
    function unlockKeyboardOnThickboxClose() {
        var targetElements = getAllElementsNotInsideContainer(window.parent.document, 'TB_iframeContent');
        _restoreKeyboardFunctionality(targetElements);
        popDialogCallStack('TB_window');
    }

/*-----------------------------------------------------*\
 * @FOCUS
 * 
 * Functions relating to setting focus or tracking keyboard navigation
 * in preparation for forcing focus change
 * 
\*-----------------------------------------------------*/

    /* Function that sets (or returns) focus to the element specified by the selector,
        * for example, when closing a dialog or modal and we return focus to the button or link clicked to open the associated dialog
        * @param selector : a jQuery CSS selector string
        * If the element is no longer present on the page or no longer found using $(selector) focus is not changed (no action taken)
    */
    function setFocusToElementFromCssSelector(selector) {
        try {
            if (selector && $(selector).length > 0) {
                let element = $(selector)[0];
                flagLastKeyclick(element);
                element.focus();
            }
        }
        catch (e) {
          // do nothing - assume page behind dialog was updated and changed significantly (selector no longer valid in current context)
        } 
    }

    /* Function that returns true if the element can take keyboard focus.
        * Returns true for all native focus elements (A, AREA, BUTTON, DETAILS, INPUT, SELECT, TEXTAREA) and elements with tabindex="0"
        * Returns false if the element is hidden (display="none" or aria-hidden="true" or type="hidden" or element.disabled is true)
        * @param element : The element in question.
    */
    function isFocusableElement(element) {
        var canFocus = false;
        if (isTabIndexSpecified(element)) {        
            canFocus = (element.getAttribute('tabindex') >= 0) ? true : false;
        } else if (isNativeFocusElement(element)) {
            canFocus = true;
        }
        return canFocus && element.style.display !== 'none'
            && element.getAttribute('aria-hidden') !== 'true'
            && element.getAttribute('type') !== 'hidden'
            && (typeof element.disabled === undefined || element.disabled !== true)
            && (isEmpty(element.id) || !element.id.startsWith('span-top-of-main-content--'))
            // ADD check for any of the CampusGroups 'hidden' classes
    }

    /* Function that finds the first actionable element (that can take keyboard focus) inside the specified container element
        * @param containerElement : the element (ex. DIV) containing the subset of elements in which focus is to be placed (ex. primary-modal)
    */
    function findFirstActionableElementInsideContainer(containerElement) {
        if (isEmpty(containerElement)) { return; } //Can this happen??

        // don't follow the tree below elements that are hidden from view
        if (containerElement.style.display === 'none' || containerElement.getAttribute('aria-hidden') === 'true') { return; }

        // don't focus on the [x] close button at the top of modal dialogs => no, let's change the design of modal to follow proper order
        if (containerElement.tagName === 'BUTTON' && containerElement.classList.toString().includes('close')) { return; }

        // don't focus on the <span id="span-top-of-main-content--*"> at the top of main content

        // IF container element itself is actionable - return it
        if (isFocusableElement(containerElement)) {
            return containerElement;
        }

        // CHECK all the child containers
        for (let i = 0; i < containerElement.children.length; i++) {
            let childElement = containerElement.children[i];
            if (childElement.tagName !== "SCRIPT" && childElement.tagName !== "STYLE") {
                if(childElement.tagName === "IFRAME"){
                    const iframeURL = new URL(childElement.src);
                    if(iframeURL.hostname === window.location.hostname && iframeURL.pathname.startsWith("/webapp/") && childElement.contentDocument.children.length > 0){
                        // Get the router-outlet inside the Angular iframe
                        let routerOutlet = childElement.contentDocument.querySelector('router-outlet');
                        // Get the element immediately after the router-outlet
                        if (routerOutlet) {
                            childElement = routerOutlet.nextElementSibling;
                        }
                    }
                }
                var focusElement = findFirstActionableElementInsideContainer(childElement);
                if (!isEmpty(focusElement)) {
                    return focusElement;
                }
            }
        }
    }

    /* Function that waits for one or more elements matching $(selector)
     * @param selector      : a valid jQuery selector string
     * @param asyncCallback : callback function, called if selector is found (passing results of $(selector))
     * @param waitMillis    : (optional) Defaults to 33; number of milliseconds to wait between query for selector
     * @param timeoutMillis : (optional) Defaults to 2000; maximum number of milliseconds to wait on successful query for selector
     *
     * If $(selector) finds one or more elements, asyncCallback() is called passing the results of the query
     * If the timeout is reached, the callback is not called
     * In both cases the interval is cleared (when found or on timeout)
     */ 
    function waitFor(selector, asyncCallback, waitMillis, timeoutMillis) {

        if (isEmpty(selector) || !isString(selector)) {
            console.error('selector is invalid');
            return;
        }
        if (isEmpty(asyncCallback) || !isFunction(asyncCallback)) {
            console.error('asyncCallback is invalid');
            return;
        }

        if (isEmpty(waitMillis) || !isNumber(waitMillis) || waitMillis < 33) {
            waitMillis = 33;            
        } else if (waitMillis > 1000) {
            waitMillis = 1000;
        }

        if (isEmpty(timeoutMillis) || !isNumber(timeoutMillis)) {
            timeoutMillis = 2000; // default 2 seconds
        } else if (timeoutMillis > 15000) {
            timeoutMillis = 15000; // max 15 seconds
        }

        let intervalId = setInterval(function() {
            if (timeoutMillis < 0) {
                clearInterval(intervalId);
            } else {
                let found = $(selector);
                if (found && found.length > 0) {
                    asyncCallback(found);
                    clearInterval(intervalId);
                }
            }
            timeoutMillis -= waitMillis;
        }, waitMillis);
    }

    /* Function that returns true if the sidebar is in the collapsed (icon) state */
    function isSidebarCollapsed() {
        return document.body.classList.contains('enlarged');
    }

    /* Function to navigate to the first menu item */
    function navigateToActiveMenuItem() {
        var activeMenuItem = $('#sidebar-menu ul li a.active').first();
        if (activeMenuItem.length > 0) {
            activeMenuItem[0].focus();
        }
    }

    /* Function that sets focus on the top left navigation menu item when the user clicks Skip to Navigation from the skip-links
        * If the left navigation is present, but hidden (ex. high-zoom or smaller browser window) we'll first expand it, but leave a focusin handler on main content to close the menu if required
    */
    function setFocusToSidebar() {
        async function callback() {
            let focusEl = getSidebarActiveElement();
            if (!isEmpty(focusEl)) {
                $(focusEl).trigger("focus");
            }
        }
        if (isSidebarCollapsed() && isAccKeyboardMode()) {
            navigateToActiveMenuItem();
        }
        waitFor('#sidebar li.active', callback);
    }

    /* Function that sets keyboard focus to the most appropriate element inside the main content area, assuming new content has been loaded
        * @param forceAccKeyboardMode : true if you want to force enabling of acc-keyboard-mode
        * forceAccKeyboardMode is used by "Skip to Main Content" which keyboard users land on before the browser tab captures any keyboard activity
        * acc-keyboard-mode is only enabled after the users presses TAB (or shift-tab) 3 or more times and so is not usually enabled when the user
        * loads a new page and chooses "Skip to Main Content" after only pressing tab once.
    */
    function setFocusToContent(forceAccKeyboardMode) {
        if (bypassFocusToMainContent) {
            bypassFocusToMainContent = false;
            return;
        }
        if (forceAccKeyboardMode === true && !isAccKeyboardMode()) { $("body").addClass("acc-keyboard-mode"); }

        var focusElement = $('.content__top-element')[0];
        if (isEmpty(focusElement)) {
            focusElement = findFirstActionableElementInsideContainer($("#page-cont")[0]);
        }
        $(focusElement).trigger("focus");
    }

    /* Function that sets keyboard focus to the first interactive element inside the modal main-content
        * (below the 'x' to close the modal in the title bar)
        * NOTE future: we should target the first interactive element that's not the 'x'
        *              but include elements in the title bar such as "Past Emails"
        * @param modalId : the ID of the modal which is to take focus (ex. primary-modal)
    */
    function setFocusToModal(modalId) {
        let pageContainer = $("#" + modalId)[0];
        let focusElement = findFirstActionableElementInsideContainer(pageContainer);
        $(focusElement).trigger("focus");
    }

    /* Function that sets focus to the first actionable element inside a container referenced by ID
        * If the element associated with containerId is not found, no action is taken
        * @param containerId : the ID of the container element (div, span, whatever) inside which focus is to be set
    */
    function setKeyboardFocusInsideContainer(containerId) {
        if (!isAccKeyboardMode()) { return; }
        let focusElement = findFirstActionableElementInsideContainer($('#' + containerId.replace('#', ''))[0]);
        if (isEmpty(focusElement)) { return; }
        focusElement.focus();
    }

/*-----------------------------------------------------*\
 * @TITLE
 * 
 * Manage the document title (set title when empty, determine appropriate title)
 * 
\*-----------------------------------------------------*/

    /* Function that sets the page title based on information defined in HeaderBootstrap (strPageName)
        * Don't call until after the page has loaded (presently only called once from setupAjaxAccessibility() above on every AJAX call successful return)
        * @global jsPageName : defined in HeaderBootstrap.ascx (based on the value assigned to VB global strPageName)
    */     
    function setPageName() {
        var isAjax = (jsAjaxRequest === false) ? false : true;
        if (isAjax && !isTransitionPageNav) {
            return;
        }
        isTransitionPageNav = false; // reset to default/false state

        var titleText = (typeof jsPageName === 'undefined') ? '' : jsPageName.trim();
        if (!isEmpty(titleText)) {
            document.title = convertToTitleCase(titleText.trim());
            setMainContentAriaLabel();
        }
    }

    /* Function that maps the current page title to the main content container (presently #page-cont)
        * The screen reader will read the page title as you enter the main content area
        * FUTURE: create a quick function to return the main content container (centralize reference and support perhaps different pages)
        *         discuss with Adrien
    */
    function setMainContentAriaLabel() {
        var documentTitle = document.title.trim();
        if (isEmpty(documentTitle)) {documentTitle = "Main Content."}
        if (!documentTitle.endsWith('.')) {documentTitle += '.';}
        $('#page-cont').attr('aria-label', documentTitle);
    }

    /* Function that sets the dialog accessibility label on the dialog container (#primary-modal or #secondary-modal with role=dialog)
     * @param modalId : the ID of the modal which is to take focus (ex. primary-modal)
    */
    function setModalDialogTitle(modalId) {
        if (isEmpty(modalId)) { console.error('modalId is empty'); return; }
        
        let titleElem = $('#' + modalId + ' .modal-header h1');
        if (titleElem.length < 1) {
            titleElem = $('#' + modalId + ' .modal-header h2');
        }
        if (titleElem.length > 0) {
            let titleText = titleElem.text();
            if (!isEmpty(titleText)) {
                $('#' + modalId).attr('aria-label', getSafeAriaLabelTextTrim(titleText));
            }
        }
    }

/*-----------------------------------------------------*\
 * @IMAGES
 * 
 * Functions to assist with accessibility requirements for images and CSS icons
 * 
\*-----------------------------------------------------*/

    /* Function that adds keyboard and screen reader support to tooltip elements.
        * @param tooltipElement : a tooltip to be configured for keyboard/screen reader usability
        * There is no actual CSS definition for .aria-tooltip (and is not required)
    */
    function setTooltipAriaLabel(tooltipElement) {
        if (!isEmpty($(tooltipElement).attr('data-original-title'))) {
            // The actual content of the tooltip will be read by the screen reader (we just want to let the user know whey're on a tooltip)
            $(tooltipElement).addClass('aria-tooltip');
            $(tooltipElement).attr('aria-label', 'Tooltip. ' + getSafeAriaLabelTextTrim($(tooltipElement).attr('data-original-title')));
            $(tooltipElement).attr('role', 'tooltip');
            $(tooltipElement).attr('tabindex', '0');
        }
    }

    /* Function that selects all tooltips and enables them for keyboard/screen reader use (will not select tooltips that are already setup or previously setup)
    */
    function setupTooltipAccessibility() {
        // TOOLTIPS need to be enabled for keyboard focus and additional screen reader context
        var elements = $('[data-toggle="tooltip"]:not(.aria-tooltip)');
        for (var i=0; i<elements.length; i++) {
            if (isEmpty(elements[i].getAttribute('aria-label'))) {
                setTooltipAriaLabel(elements[i]);
            }
        }
    }

    /* Function that hides an element from the keyboard and screen reader but leaves the element visible on the UI (as original)
        * @param element : the element to be hidden from keyboard/screen reader
        * There is no actual CSS definition for .aria-decorative (and is not required)
    */
    function hideElementFromKeyboardAndScreenReader(element) {
        // quick exit for elements previously processed
        if ($(element).hasClass('aria-decorative')) { return; }

        // do the work for new elements
        $(element).addClass('aria-decorative');
        $(element).attr('role', 'presentation');
        $(element).attr('aria-hidden', 'true');
        if (element.tagName === 'IMG') {
            $(element).attr('alt', '');
        }
        if (isTabIndexSpecified(element) && $(element).attr('tabindex') >= 0) {
            $(element).attr('tabindex', '-1');
        }
    }

    /* Function that hides decorative images from the keyboard and screen reader
    */
    function hideDecorativeImagesAndIconsFromScreenReader() {
        let selectors = ['.mdi', 'img[src^="/images/"]', '.caret', '.glyphicon'];

        // we'll concatonate all elements of interest into a single array, then do the work
        var elements = []; 
        for (var i=0; i<selectors.length; i++) {
            elements = elements.concat($(selectors[i] + ':not(.aria-decorative)').toArray());
        }

        for (var i=0; i<elements.length; i++) {
            let element = elements[i];

            var shouldBeAriaHidden = true;
            if ($(element).attr('data-toggle') === 'tooltip') {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).attr('onclick'))) {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).attr('onerror'))) {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).attr('data-keyclick'))) {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).text())) {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).attr('alt')) && $(element).attr('aria-decorative') === 'false') {
                shouldBeAriaHidden = false;
            } else if (isTabIndexSpecified(element) && $(element).attr('tabindex') >= 0) {
                shouldBeAriaHidden = false;
            } else if (!isEmpty($(element).attr('src'))) {
                if ($(element).attr('src').includes('/images/revisions_grey.png')) {
                    shouldBeAriaHidden = false;
                }
            }

            if (shouldBeAriaHidden) {
                hideElementFromKeyboardAndScreenReader(element);
            }
        }
    }

/*-----------------------------------------------------*\
 * @SAVE-FORM-ERRORS
 * 
 * Manage error presentation when saving forms and errors are found
 * 
\*-----------------------------------------------------*/

    /* --------------------------------------------------------------------------------------------------
     * SAVE FORM Errors - handle errors on 'Save' form (for forms built in table_bootstrap__.inc)
     * 
     */
    var saveFormErrorCount = 0;
    var saveFormErrorMessages = {};

    /* --------------------------------------------------------------------------------------------------
     * Returns true if errors were encountered when saving the form.
     * 
     */
    function hasSaveFormErrors() {
        return (saveFormErrorCount > 0) ? true : false;
    }

    /* --------------------------------------------------------------------------------------------------
     * Clear the save form error history.
     * 
     */
    function clearSaveFormErrors() {
        saveFormErrorCount = 0;
        saveFormErrorMessages = {};
    }

    /* --------------------------------------------------------------------------------------------------
     * Add an error to saveFormErrorMessages.
     * - These will be presented to the user at the top of the form when save fails.
     * 
     * @param {Element} obj     : The input or equivalent that encountered an error on attempting to save the form     
     * @param {string}  message : The error message associated with the error encountered.
     * 
     */
    function addSaveFormError(obj, message) {

        // @REMOVE-FROM here
        if (isEmpty(obj)) {
            console.error('obj is empty');
            return;
        }

        if (isEmpty(message)) {
            console.error('message is empty');
            return;
        } else if (!isString(message)) {
            console.error('message is not a string');
            return;
        }
        // @REMOVE-TO here

        saveFormErrorCount++;

        // @INVESTIGATE clean this up a bit below here
        var labelText = getLabelTextForElementWithoutAsterix(obj);
        if (isEmpty(labelText)) {
            console.error('labelText is empty');
            return;
        }

        if (!isEmpty(saveFormErrorMessages[labelText])) {
            // console.error('> addSaveFormError() labelText=' + labelText + ' already exists');
            saveFormErrorCount--;
            return;
        }

        saveFormErrorMessages[labelText] = message;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns a string representing the UI label associated with an input that received an error when
     * attempting to save the form. The string has a comma ',' or period '.' appended, depending on whether
     * or not it's the last error found while saving.
     * 
     * Used to format the error notification (and associated aria content) when errors are found.
     * 
     * @param {string}  labelText  : The UI label for the input that encountered an error on save.
     * @param {boolean} isLast     : True if this is the last error encountered while saving the form.
     * 
     */
    function getSaveFormErrorFieldDescription(labelText, isLast) {

        // @REMOVE-FROM here
        if (isEmpty(labelText)) {
            console.error('labelText is empty');
            return '';
        }

        if (isEmpty(isLast)) {
            console.error('isLast is empty');
            return '';
        }   
        // @REMOVE-TO here

        if (isLast) {
            labelText += '.';
        } else {
            labelText += ',';
        }

        if (labelText.startsWith('*')) {
            labelText = labelText.substring(1).trim();
        }
        labelText += ' ';

        return labelText;
    }

    /* --------------------------------------------------------------------------------------------------
     * Generate an HTML insert with details on errors encountered when attempting to save a form.
     * - The HTML will be inserted into the '#table_form_save_errors' container at the top of the page
     * - Styled after the existing error messages at the top of main content
     * 
     */
    function displaySaveFormErrors() {
        // console.log('> displaySaveFormErrors() begins...');

        var topFormErrorMessageDiv = document.getElementById('table_form_save_errors');
        if (isEmpty(topFormErrorMessageDiv)) {
            console.error('topFormErrorMessageDiv is empty.');
            return;
        }

        if (!hasSaveFormErrors()) {
            topFormErrorMessageDiv.style.display = 'none';
            return;
        }

        var errorHtml = '<i class="mdi mdi-block-helper"></i>';
        if (saveFormErrorCount == 1) {
            errorHtml += 'Save failed. The following field is empty or contains invalid data: ';
        } else {
            errorHtml += 'Save failed. The following ' + saveFormErrorCount + ' fields are empty or contain invalid data: ';
        }

        var count = 0;
        var errorFieldsHtml = '';
        for (var fieldName in saveFormErrorMessages) {
            var isLast = false;
            if (count >= (saveFormErrorCount - 1)) {
                isLast = true;
            }
            // console.log('> fieldName=' + fieldName + ', isLast=' + isLast);
            errorFieldsHtml += getSaveFormErrorFieldDescription(fieldName, isLast);
            count++;
        }

        if (count < saveFormErrorCount)
        {
            var moreFields = null;
            var moreFieldsCount = saveFormErrorCount - count;
            if (moreFieldsCount === 1) {
                moreFields = ' and 1 other field';
            } else {
                moreFields = ' and ' + moreFieldsCount + ' other fields';
            }
            errorFieldsHtml += getSaveFormErrorFieldDescription(moreFields, true);
        }

        if (isEmpty(errorFieldsHtml)) {
            console.error('errorFieldsHtml is empty');
            errorFieldsHtml = '';
        }
        errorHtml += getEncodedHtmlContent(errorFieldsHtml);

        topFormErrorMessageDiv.innerHTML = errorHtml;
        topFormErrorMessageDiv.style.display = '';
        topFormErrorMessageDiv.focus();
    }

/*-----------------------------------------------------*\
 * @UTILITIES
 * 
 * Basic utility or helper functions
 * 
\*-----------------------------------------------------*/

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a bigint object.
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isBigInt(x) {
        return typeof x === 'bigint';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a boolean object.
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isBoolean(x) {
        return typeof x === 'boolean';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a JavaScript Date object.
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isDate(x) {
        if (isEmpty(x)) {
            return false;
        } else if (!isObject(x)) {
            return false;
        } else if (isEmpty(x.now)) {
            return false;
        } else if (isEmpty(x.getTime)) {
            return false;
        } else if (isEmpty(x.getYear)) {
            return false;
        } else if (isEmpty(x.getMonth)) {
            return false;
        } else if (isEmpty(x.UTC)) {
            return false;
        }
        return true;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x has no value (undefined, null or an empty trim() string)
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isEmpty(x) {

        if (typeof x === 'undefined') {
            return true;
        } else if (x === null) {
            return true;
        } else if (typeof x === 'string' && x.trim().length < 1) {
            return true;
        }
        return false;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a function object
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isFunction(x) {
        return typeof x === 'function';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a hex character
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    var ulcHexCharacters = '0123456789abcdefABCDEF';
    function isHexCharacter(c) {
        if (isEmpty(c)) {
            return false;
        } else if (!isString(c)) {
            return false;
        }

        if (!ulcHexCharacters.includes(c)) {
            return false;
        }
        return true;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a hex string
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isHexString(s) {
        if (isEmpty(s)) {
            return false;
        } else if (!isString(s)) {
            return false;
        }

        for (var i = 0; i < s.length; i++) {
            if (!isHexCharacter(s.charAt(i))) {
                return false;
            }
        }

        return true;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a jQuery object
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isJQueryElement(x) {
        if (isEmpty(x)) {
            return false;
        } else if (!isEmpty(x.jquery)) {
            return true;
        }
        return false;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is null
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isNull(x) {
        if (isUndefined(x)) {
            console.error('x is undefined');
            return;
        }
        return x === null;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a number
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isNumber(x) {
        return typeof x === 'number' && !isNaN(x);
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a boolean number value (0 or 1)
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isNumberBoolean(x) {
        // return true if x is a number and is either zero or one
        if (!isNumber(x)) {
            return false;
        }
        return x === 0 || x === 1;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is an object
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isObject(x) {
        return typeof x === 'object';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a primitive JavaScript data type
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isPrimitive(x) {
        //
        // return true if x is a JavaScript primitive data type
        //
        if (isBigInt(x)) {
            return true;
        } else if (isBoolean(x)) {
            return true;
        } else if (isFunction(x)) {
            // functions are treated as primitives in this implementation
            return true;
        } else if (isNumber(x)) {
            return true;
        } else if (isString(x)) {
            return true;
        } else if (isSymbol(x)) {
            return true;
        } else if (isUndefined(x)) {
            return true;
        }

        return false;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a string
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isString(x) {
        return typeof x === 'string';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is an boolean string value ('true' or 'false')
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isStringBoolean(x) {
        // return true if x is a string with value 'true' or 'false' (after trim() and toLowerCase() applied)
        if (!isString(x)) {
            return false;
        }
        x = x.trim().toLowerCase();
        return x === 'true' || x === 'false';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a symbol
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isSymbol(x) {
        return typeof x === 'symbol';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is undefined
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isUndefined(x) {
        return typeof x === 'undefined';
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns true if x is a string representing a UUID value
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function isUUID(uuidString) {
        // a hack implementation but usable
        if (isEmpty(uuidString)) {
            return false;
        } else if (!isString(uuidString)) {
            return false;
        } else if (uuidString.length !== 36) {
            return false;
        } else if (uuidString.charAt(8) !== '-') {
            return false;
        } else if (uuidString.charAt(13) !== '-') {
            return false;
        } else if (uuidString.charAt(18) !== '-') {
            return false;
        } else if (uuidString.charAt(23) !== '-') {
            return false;
        } else {
            uuidString = uuidString.trim().toLowerCase().replaceAll('-', '');
            if (uuidString.length != 32) {
                return false;
            }
            if (!isHexString(uuidString)) {
                return false;
            }
        }
        return true;
    }

    /* --------------------------------------------------------------------------------------------------
     * Toggle a boolean value (value may be represented as type boolean, a number (0/1) or a string (true/false))
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function toggleBoolean(x) {
        if (isEmpty(x)) {
            console.error('x is empty');
            return;
        } else if (isBoolean(x)) {
            return !x;
        } else if (isStringBoolean(x)) {
            return (x === 'false') ? 'true' : 'false';
        } else if (isNumberBoolean(x)) {
            return (x === 0) ? 1 : 0;
        } else {
            console.error('typeof x is unsupported');
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns a string used for console log indentation.
     * - Used by getDescription() to generate easy to read console output.
     * 
     * @param {number} indentCount  : indentation level (using spaces)
     * 
     */
    function getIndent(indentCount) {
        var indent = '';
        for (var count = 0; count < indentCount; count++) {
            indent += ' ';
        }
        return indent;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns a string used for console log indentation.
     * - Used by getDescription() to generate easy to read console output.
     * 
     * @param {object} x            : value/variable that you want to describe
     * @param {number} indentCount  : indentation level (using spaces; used for recursive calls while building description)
     * 
     */
    var newlineChar = '\n';
    function getDescription(x, indentCount) {
        var indentCountMax = 8;

        if (isEmpty(indentCount)) {
            indentCount = 0;
        }

        if (isUndefined(x)) {
            return '{undefined}';
        } else if (isNull(x)) {
            return '{null}';
        } else if (isUUID(x)) {
            return '{UUID}' + x.toString();
        } else if (typeof x === 'string') {
            return '{string}' + x.toString();
        } else if (typeof x === 'number') {
            return '{number}' + x.toString();
        } else if (typeof x === 'boolean') {
            return '{boolean}' + x.toString();
        } else if (typeof x === 'symbol') {
            return '{symbol}' + x.toString();
        } else if (typeof x === 'function') {
            return '{function}' + x.toString();
        } else if (isDate(x)) {
            return '{date}' + x.toString();
        } else if (typeof x === 'object') {
            if (!isEmpty(x.length)) {
                var description = '{object,length=' + x.length + '}' + newlineChar;
                indentCount += 2;
                if (indentCount > indentCountMax) {
                    description += getIndent(indentCount) + 'index[' + index + ']={more...;indentCount>indentCountMax}' + newlineChar;
                } else {
                    for (var index = 0; index < x.length; index++) {
                        description += getIndent(indentCount) + 'index[' + index + ']=' + getDescription(x[index], indentCount) + newlineChar;
                    }
                    return description.trim();
                }
            } else if (getPropertyCount(x) > 0) {
                var description = '{object,propertyCount=' + getPropertyCount(x) + '}' + newlineChar;
                indentCount += 2;
                if (indentCount > indentCountMax) {
                    description += getIndent(indentCount) + 'index[' + index + ']={more...;indentCount>indentCountMax}' + newlineChar;
                } else {
                    for (var property in x) {
                        description += getIndent(indentCount) + 'property[' + property + ']=' + getDescription(x[property], indentCount) + newlineChar;
                    }
                    return description.trim();
                }
            }
            return '{object}' + x.toString();
        } else {
            var unknownType = typeof x;
            return '{unknown-type[' + unknownType + ']}' + x.toString();
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns the count of the number of properties the object 'x' has.
     * 
     * @param {Object} x  : value to be considered
     * 
     */
    function getPropertyCount(x) {
        if (isEmpty(x)) {
            return 0;
        } else if (isPrimitive(x)) {
            return 0;
        }

        var count = 0;
        for (k in x) {
            count++;
        }

        return count;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns a partial DOM path for the element specified.
     * - used for debug/diagnostics to get details on the path of the element being inspected
     * - called by getElementPath(elem) when building the element's full path
     * 
     * @param {Element} elem  : an element in the document
     * 
     */
    function getElementPathPart(elem) {
        if (isEmpty(elem)) {
            return '';
        }
        var pathPart = elem.tagName;
        if (!isEmpty(elem.id)) {
            pathPart += '#' + elem.id;
        } else if (!isEmpty(elem.classList) && !isEmpty(elem.classList.toString())) {
            pathPart += '.' + replaceAll(elem.classList.toString(), ' ', '.');
        }
        return pathPart;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns the full document path for an element.
     * - used for debug/diagnostics to get details on the path of the element being inspected
     * 
     * @param {Element} elem  : an element in the document
     * 
     */
    function getElementPath(elem) {
        // @REMOVE-FROM here
        if (isEmpty(elem)) {
            console.error('elem is empty');
            return;
        }
        // @REMOVE-TO here

        const pathPart = getElementPathPart(elem);

        var parentPath = '';
        const parentElement = elem.parentElement;
        if (!isEmpty(parentElement)) {
            parentPath = getElementPath(parentElement);
        }
        var dot = '';
        if (parentPath.length > 0) {
            dot = '.';
        }

        return parentPath + dot + pathPart;
    }

    /* --------------------------------------------------------------------------------------------------
     * Replace all occurances of 'x' with 'y' in a string.
     * 
     * @param {string} x  : the string to be replaced
     * @param {string} y  : the replacement text string
     * 
     */
    function replaceAll(text, x, y) {
        if (text === undefined || text === null || text === '') { return text; }
        if (x === undefined || x === null || x === '') { return text; }
        if (!text.includes(x)) { return text; }
        if (y === undefined || y === null) { y = ''; }
        return text.split(x).join(y);
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns the string with the first letter capitalized.
     * 
     * @param {string} s  : A string value in which we want to capitalize the first letter.
     * 
     */
    function firstToCap(s) {
        return s.charAt(0).toUpperCase() + s.slice(1);
    }

    /* --------------------------------------------------------------------------------------------------
     * Convert a string to title case, with each word in the title capitalized.
     * 
     * @param {string} text  : A text string to be converted to title case.
     * 
     */
    function convertToTitleCase(text) {
        if (isEmpty(text)) {
            console.error('text is empty');
            return '';
        }
        if (text.length === 0) {
            return '';
        }
        var parts = text.split(' ');
        for (var i=0; i<parts.length; i++) {
            if (parts[i].length < 2) {
                parts[i] = parts[i].toUpperCase();
            } else {
                var firstChar = parts[i].charAt(0);
                if (firstChar === "'" || firstChar === '"' || firstChar === '(' || firstChar === '[' || firstChar === '{' || firstChar === '<' || firstChar === '$') {
                    if (parts[i].length < 3) {
                        parts[i] = firstChar + parts[i].charAt(1).toUpperCase();
                    } else {
                        parts[i] = firstChar + parts[i].charAt(1).toUpperCase() + parts[i].substring(2);
                    }
                } else {
                    parts[i] = parts[i].charAt(0).toUpperCase() + parts[i].substring(1);
                }
            }       
        }
        return parts.join(' ');
    }

    /* --------------------------------------------------------------------------------------------------
     * Return the number of seconds (to 3 decimals) elapsed since the specified epoch value (in millis).
     * 
     * @param {number} epochStart  : An epoch value, in milliseconds.
     * 
     */
    function getElapsedSeconds(epochStart) {
        if (isEmpty(epochStart)) {
            console.error('epochStart is empty');
            return;
        }
        return (Date.now() - epochStart) / 1000.0;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns an HTML encoded string equivalent to text when decoded. Used when adding database (or any
     * potentially unsafe text) as HTML to the document.
     * 
     * @param {string} text  : The text string to be HTML encoded.
     * 
     */
    function getEncodedHtmlContent(text) {
        if (isEmpty(text)) {
            return '';
        }

        var elem = document.createElement('span');
        elem.textContent = text;

        var encodedHtml = elem.innerHTML;
        if (isEmpty(encodedHtml)) {
            encodedHtml = '';
        }

        return encodedHtml;
    }


/*-----------------------------------------------------*\
 * @ARIA-LABEL
 * 
 * Functions to prepare and set aria-label or alt= attribute text (or any attribute text enclosed in single or double quotes)
 * 
\*-----------------------------------------------------*/

    /* Function that adds screen reader context (aria-label) to entries in the topbar notification dropdown.
     * The notifications are pre-generated and stored in the database so we can only update/alter them after being rendered in the DOM
     * The notifications are loaded only if you open the dropdown or click "load more" inside the dropdown
     * Using setTimeout() to make sure all associated DOM elements have had time to be rendered
     */
    function setupNotificationsDropdownAccessibility() {
        setTimeout(function() {
            let notificationElements = $('#header__notif-dropdown #all_notifications .notification-cg--details');
            for (var i=0; i<notificationElements.length; i++) {
                let notificationElement = notificationElements[i];
                let notificationDetails = addAriaLabelPunctuation(getSafeAriaLabelTextTrim(notificationElement.innerHTML));
                let notificationLinks = $(notificationElement).find('a');

                // the links embedded within the notification content
                for (var j=0; j<notificationLinks.length; j++) {
                    let notificationLink = notificationLinks[j];
                    let notificationLinkAriaLabel = $(notificationLink).attr('aria-label');
                    if (isEmpty(notificationLinkAriaLabel)) {
                        let notificationLinkHref = $(notificationLink).attr('href');
                        let notificationLinkDescription = 'Click for more details.';
                        if (notificationLinkHref.includes('/student_profile?uid')) {
                            notificationLinkDescription = "Click to open the user's profile.";
                        } else if (notificationLinkHref.includes('/d?t=event')) {
                            notificationLinkDescription = "Click to open the event page.";
                        } else if (notificationLinkHref.includes('/rsvp?id')) {
                            notificationLinkDescription = "Click to open the event page.";
                        } else if (notificationLinkHref.includes('/officer_login_redirect?r=')) {
                            notificationLinkDescription = "Click to open group dashboard.";
                        }
                        $(notificationLink).attr('aria-label', notificationDetails + ' ' + notificationLinkDescription);
                        $(notificationLink).attr('data-a11y', 'a01');
                    }
                }

                // 'x' button to clear/delete the notification from dropdown
                let deleteNotificationLink = $(notificationElement).parent().find('a.close_sugg');
                if (deleteNotificationLink.length === 1) {
                    if (isEmpty($(deleteNotificationLink[0]).attr('aria-label'))) {
                        $(deleteNotificationLink[0]).attr('aria-label', 'Clear notification');
                        $(deleteNotificationLink[0]).attr('aria-description', notificationDetails);
                    }
                }

                // Notification icon link (left of notification message); sometimes just an <img> sometimes an <a> with embedded <img>
                let notificationIconLinks = $(notificationElement).closest('tr').find('> td > a');
                if (notificationIconLinks.length === 1) {
                    $(notificationIconLinks[0]).attr('aria-label', 'Notification icon');
                    $(notificationIconLinks[0]).attr('aria-description', 'Notification icon. ' + notificationDetails + " Click to view user's profile.");
                }

                $(notificationElement).attr('data-a11y', 'a01'); // flag content OK for automated testing
            }
        }, 100);
    }

    /* Function that extracts the first occurance of text inside two markers (x and y)
     * If found, the entire text block, including the opening and closing markers, is returned
     * @param text : String text in which we want to find any text blocks starting with 'x' and ending with 'y'
     * @param x    : String opening marker, for example, '<script>'
     * @param y    : String closing marker, for example, '</script>'
     * @param returnFullTailIfYNotFound : when true and 'x' is found but 'y' is not, the entire string starting from 'x' is returned
     */
    function extractFirstBlockInsideMarkers(text, x, y, returnFullTailIfYNotFound) {
        if (isEmpty(text)) { return; } 
        if (isEmpty(x)) { return; }
        if (isEmpty(y)) { return; }
        if (isEmpty(returnFullTailIfYNotFound)) { returnFullTailIfYNotFound = false; }

        var lcText = text.toLowerCase();
        var sx = lcText.indexOf(x.toLowerCase());
        if (sx < 0) { return; }

        var ex = lcText.indexOf(y.toLowerCase(), sx + 1);
        if (ex < 0) {
            if (returnFullTailIfYNotFound) {
                return text.substring(sx);
            } else {
                return;
            }
        }
        return text.substring(sx, ex + y.length);
    }

    /* Function that returns the index of the next occurance of the text 'searchFor' within the string 'text', starting from index. Search is case insensitive.
     * @param text      : text within which to find 'searchFor'
     * @param searchFor : the string to find in 'text'
     * @param index     : the starting index to begin searching
     * 
     * Returns -1 if the string 'searchFor' is not found in 'text'.
     */
    function indexOfIgnoreCase(text, searchFor, index) {
        if (typeof text === 'undefined' || text === null) { return -1; }
        if (typeof searchFor === 'undefined' || searchFor === null) { return -1; }
        if (isEmpty(index)) { index = 0; }
        return text.toLowerCase().indexOf(searchFor.toLowerCase(), index);
    }

    /* Function that returns the index of the next HTML open quote (single quote or double-quote) from the specified index
     * @param htmlContent : text content that may or may not contain HTML (possibly with <script>, <style> or VB.Net blocks, etc.)
     * @param sx          : start index to begin scanning forward (no default value; you must pass 0 (zero) to start from the beginning)
     */
    function getNextHtmlOpenQuoteIndex(htmlContent, sx) {
        if (isEmpty(htmlContent)) { return -1; }
        if (isEmpty(sx) || sx < 0 || sx >= htmlContent.length) { return -1; }

        var ix1 = htmlContent.indexOf('"', sx); // next double-quote
        var ix2 = htmlContent.indexOf("'", sx); // next single-quote
        if (ix1 >= sx && ix2 >= sx) {
            return Math.min(ix1, ix2);
        } else if (ix1 >= sx) {
            return ix1;
        } else if (ix2 >= sx) {
            return ix2;
        }
        return -1;
    }

    /* Function that returns the index of the closing quote for a given open quote within HTML content, starting at the specified index
     * @param htmlContent    : text content that may or may not contain HTML (possibly with <script>, <style> or VB.Net blocks, etc.)
     * @param openQuoteIndex : index within htmlContent of the opening quote for which we want to find the associated closing quote
     */
    function getNextHtmlCloseQuoteIndex(htmlContent, openQuoteIndex) {
        if (isEmpty(htmlContent)) { return -1; }
        if (isEmpty(openQuoteIndex) || openQuoteIndex < 0 || openQuoteIndex >= htmlContent.length) { return -1; }

        const chQuote = htmlContent[openQuoteIndex];
        var ixOnlyQuote = htmlContent.indexOf(chQuote, openQuoteIndex + 1);
        var ixEscapeQuote = htmlContent.indexOf('\\' + chQuote, openQuoteIndex + 1);
        while (ixOnlyQuote < htmlContent.length && ixOnlyQuote === (ixEscapeQuote + 1)) {
            ixOnlyQuote = htmlContent.indexOf(chQuote, ixOnlyQuote + 1);
            ixEscapeQuote = htmlContent.indexOf('\\' + chQuote, ixOnlyQuote + 1);            
        }
        return (ixOnlyQuote > openQuoteIndex) ? ixOnlyQuote : -1;
    }

    /* Function that returns true if the specified character is a valid HTML tag identifier (typically upper/lower a-z but also !, %, etc.)
     * @param ch : the first character inside the associated tag (<{ch]...> OR </{ch}...>)
     */
    function isValidHtmlTagNameCharacter(ch) {
        if (isEmpty(ch) || !isString(ch)) { return false; }
        if (ch.length > 1) { ch = ch[0]; }
        if (ch >= 'A' && ch <= 'Z') return true;
        if (ch >= 'a' && ch <= 'z') return true;
        if (ch > ' ' && ch < '+') return true;
        if (ch === '/') return true;
        return false;
    }

    /* Function that returns the index of the next '<' character representing the opening of an HTML tag (or equivalent such as VB.Net < followed by %)
     * @param htmlContent : text content that may or may not contain HTML (possibly with <script>, <style> or VB.Net blocks, etc.)
     * @param sx          : start index swithin htmlContent to begin scanning forward (defaults to zero)
     */
    function getNextTagOpenIndex(htmlContent, sx) {
        if (isEmpty(htmlContent)) { return -1; }
        if (isEmpty(sx)) {
            sx = 0;
        } else if (sx < 0 || sx >= htmlContent.length) {
            return -1;
        }

        var ix = htmlContent.indexOf('<', sx);
        while (ix >= sx && ix < (htmlContent.length - 1) && !isValidHtmlTagNameCharacter(htmlContent[ix+1])) {
            ix = htmlContent.indexOf('<', ix+1);
        }
        return (ix < sx || ix >= htmlContent.length) ? -1 : ix;
    }

    /* Function that returns the index of the next '>' character representing the closing of an HTML tag
     * @param htmlContent  : text content that may or may not contain HTML (possibly with <script>, <style> or VB.Net blocks, etc.)
     * @param openTagIndex : index within htmlContent where the HTML tag was opened (index of '<' that opened the HTML tag)
     */
    function getNextTagCloseIndex(htmlContent, openTagIndex) {
        if (isEmpty(htmlContent)) { return -1; }
        if (isEmpty(openTagIndex) || openTagIndex < 0 || openTagIndex >= htmlContent.length) { return -1; }

        var closeTagIndex = htmlContent.indexOf('>', openTagIndex);
        if (closeTagIndex < 0) {return -1; }

        var openQuoteIndex = getNextHtmlOpenQuoteIndex(htmlContent, openTagIndex);
        while (openQuoteIndex > openTagIndex && openQuoteIndex < closeTagIndex) {
            var closeQuoteIndex = getNextHtmlCloseQuoteIndex(htmlContent, openQuoteIndex);
            if (closeQuoteIndex < 0) {
                return -1;
            } else if (closeQuoteIndex < closeTagIndex) {
                var oqi = getNextHtmlOpenQuoteIndex(htmlContent, closeQuoteIndex+1);
                if (oqi > closeTagIndex) {
                    return closeTagIndex;
                }
                openQuoteIndex = oqi;
            } else if (closeQuoteIndex > closeTagIndex) {
                closeTagIndex = htmlContent.indexOf('>', closeQuoteIndex);
                if (closeTagIndex < 0) {
                    return -1;
                }
                openQuoteIndex = getNextHtmlOpenQuoteIndex(htmlContent, openTagIndex);
            }
        }
        return closeTagIndex;
    }

    /* Function that returns the index of the next '</' character sequence representing the opening of a "close tag" ex. </div> or </span>, etc.
     * @param htmlContent : text content that may or may not contain HTML (possibly with <script>, <style> or VB.Net blocks, etc.)
     * @param sx          : start index swithin htmlContent to begin scanning forward (defaults to zero)
     */
    function getNextClosingTagIndex(htmlContent, sx) {
        if (isEmpty(htmlContent)) { return -1; }
        if (isEmpty(sx)) {
            sx = 0;
        } else if (sx < 0 || sx >= htmlContent.length) {
            return -1;
        }

        var ix = htmlContent.indexOf('</');
        while (ix >= sx && ix < (htmlContent.length-2) && !isValidHtmlTagNameCharacter(htmlContent[ix+2])) {
            ix = htmlContent.indexOf('</', ix+1);
        }
        return (ix < sx || ix >= htmlContent.length) ? -1 : ix;
    }

    /* Function that converts any coded character values back to the associated character.
     * @param text : string possibly containing HTML encoded characters
     */
    function decodeHtmlCodes(text) {
        let sx = text.indexOf('&#');
        while (sx >= 0) {
            let ex = text.indexOf(';', sx + 2);
            if ((ex-sx > 2) && (ex-sx <= 9)) {
                let ix = sx + 2;
                let isHtmlCode = true;
                while (isHtmlCode && ix < ex) {
                    isHtmlCode = (text[ix] >= '0' && text[ix] <= '9') ? true : false;
                    ix++;
                }
                if (isHtmlCode) {
                    let htmlCode = text.substring(sx, ex + 1);
                    let codeValue = parseInt(text.substring(sx + 2, ex));
                    text = text.split(htmlCode).join(String.fromCharCode(codeValue));
                }
            }
            sx = text.indexOf('&#', sx + 1);
        }
        return text;
    }

    /* Function that extracts pure (visible) text content from HTML content, leaving breathers. Intended for safe HTML attribute text and clear/understandable text for screen readers.
     * Extract the "usable" text content from the innerHTML of a div, span, etc.
     * 1. Remove any script or style blocks
     * 2. Append a space before any break, paragraph, span, div, etc. so the text isn't concatenated (leave space between text where we would see breaks/spaces)
     * 3. Replace tabs and double-spaces with single spaces
     * @param htmlContent : HTML content string to be converted to human visible text (if one were looking at the UI) including breathers for screen reader to read properly
     */
    function extractPureTextContentFromHtml(htmlContent) {
        if (isEmpty(htmlContent)) return ' ';

        // Specific HTML code replacement
        htmlContent = replaceAll(htmlContent, '&#171;', '<');
        htmlContent = replaceAll(htmlContent, '&#187;', '>');
        htmlContent = replaceAll(htmlContent, '&#60;', '<');
        htmlContent = replaceAll(htmlContent, '&#62;', '>');

        htmlContent = decodeHtmlCodes(htmlContent);

        // convert non-breaking space to regular space (the decoded nbsp is ascii 160)
        htmlContent = replaceAll(htmlContent, '&para;', ' ');
        htmlContent = replaceAll(htmlContent, '&nbsp;', ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(9), ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(10), ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(11), ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(13), ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(160), ' ');
        htmlContent = replaceAll(htmlContent, String.fromCharCode(182), ' ');
        htmlContent += ' '; // why again - do we need padding on the tail to help the below work better ?

        // VB.Net
        var openTag = '<%--';
        var closeTag = '--%>';
        var blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        while (!isEmpty(blockToRemove)) {
            htmlContent = htmlContent.replace(blockToRemove, ' ');
            blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        }

        // VB.net
        openTag = '<%';
        closeTag = '%>';
        blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        while (!isEmpty(blockToRemove)) {
            htmlContent = htmlContent.replace(blockToRemove, ' ');
            blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        }

        // HTML
        openTag = '<!--';
        closeTag = '-->';
        blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        while (!isEmpty(blockToRemove)) {
            htmlContent = htmlContent.replace(blockToRemove, ' ');
            blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, true);
        }

        // HTML names
        htmlContent = replaceAll(htmlContent, '&amp;', '&');
        htmlContent = replaceAll(htmlContent, '&apos;', "’");
        htmlContent = replaceAll(htmlContent, '&copy;', ' copyright ');
        htmlContent = replaceAll(htmlContent, '&deg;', ' degrees ');
        htmlContent = replaceAll(htmlContent, '&divide;', ' divided by ');
        htmlContent = replaceAll(htmlContent, '&euro;', ' euros ');
        htmlContent = replaceAll(htmlContent, '&gt;', '>');
        htmlContent = replaceAll(htmlContent, '&laquo;', '<');
        htmlContent = replaceAll(htmlContent, '&lt;', '<');
        htmlContent = replaceAll(htmlContent, '&middot;', '-');                     // minus sign
        htmlContent = replaceAll(htmlContent, '&ndash;', '-');
        htmlContent = replaceAll(htmlContent, '&quot;', '"');                       // double-quote
        htmlContent = replaceAll(htmlContent, '&raquo;', '>');
        htmlContent = replaceAll(htmlContent, '&reg;', ' registered trade mark ');
        htmlContent = replaceAll(htmlContent, '&times;', ' x ');                    // letter x
        htmlContent = replaceAll(htmlContent, '&uml;', '"');                        // double-quote

        // Script, style, etc. blocks
        const extractFullTailWhenCloseTagIsAbsent = true;
        const fullBlockRemovalTags = [ 'script', 'style', 'svg', 'math', 'meta', 'metadata', 'head', 'body', 'html', 'iframe' ];
        for (var i=0; i<fullBlockRemovalTags.length; i++) {
            let openTags = [ '<' + fullBlockRemovalTags[i] + '>', '<' + fullBlockRemovalTags[i] + ' ' ];
            let closeTag = '</' + fullBlockRemovalTags[i] + '>';

            for (var otx = 0; otx < openTags.length; otx++) {
                let openTag = openTags[otx];
                var blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, extractFullTailWhenCloseTagIsAbsent);
                while (!isEmpty(blockToRemove)) {
                    htmlContent = htmlContent.replace(blockToRemove, ' ');
                    blockToRemove = extractFirstBlockInsideMarkers(htmlContent, openTag, closeTag, extractFullTailWhenCloseTagIsAbsent);
                }
            }
        }

        // add breather spacing between tags
        htmlContent = replaceAll(htmlContent, '><', '> <').trim(); 

        // Remove just the tags (leaving inner text content)
        var otix = getNextTagOpenIndex(htmlContent);
        while (otix >= 0) {
            var ctix = getNextTagCloseIndex(htmlContent, otix);
            if (ctix > otix) {
                // console.log('> EXTRACT otix=' + otix + ' ctix=' + ctix + ' extract=[' + htmlContent.substring(otix, ctix + 1) + ']');
                htmlContent = htmlContent.substring(0, otix) + ' ' + htmlContent.substring(ctix + 1);
                otix = getNextTagOpenIndex(htmlContent, otix);
            } else {
                // an open tag '<' with no matching close to the tag (can't find '>' beyond '<')
                otix = -1;
            }
        }

        return htmlContent.trim();
    }

    /* Function that returns safe text for use in ALT attributes, aria-label attributes or other text attributes
     * Replaces all single and double quote characters with their extended ASCII code equivalents, so they function as expected using screen reader
     * Converts single and double quotes to alternate character codes that "look" the same but are not recognized as single/double quotes by HTML parser
     * @param text : String text to be prepared for assignment to an attribute for screen reader use (alt, aria-label attributes, etc.)
     * 
     * Return value will not include single or double quotes and will not incude any of the following: & < > = (+more, see below)
     */
    function getSafeAriaLabelText(text) {
        if (isEmpty(text)) { return ' '; }

        // console.log('> getSafeAriaLabelText() text=' + text);

        // Remove any HTML, script/style blocks, etc.
        var safeAttributeText = extractPureTextContentFromHtml(text);
        // safeAttributeText = unescape(safeAttributeText);

        // Replace specific characters by code
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(35), ' number sign ');  // #
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(42), ' asterix ');      // *
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(92), ' backslash ');    // backslash key
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(169), ' copyright ');   // &copy;
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(183), '-');             // &middot;
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(215), ' times ');           // &times;
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(247), ' divided by');   // &divide;
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(8211), '-');            // en dash
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(8212), '-');            // em dash
        safeAttributeText = replaceAll(safeAttributeText, String.fromCharCode(8482), ' trade mark '); // TM symbol

        // Specific to date/time text
        safeAttributeText = replaceAll(safeAttributeText, "GMT-", "GMT minus ");
	
        // Improve readability
        safeAttributeText = replaceAll(safeAttributeText, "!=", " not equal to ");
        safeAttributeText = replaceAll(safeAttributeText, "<=", " less than or equal to ");
        safeAttributeText = replaceAll(safeAttributeText, ">=", " greater than or equal to ");
        safeAttributeText = replaceAll(safeAttributeText, "<>", " less than or greater than ");

        // Safe readable text (these would otherwise be encoded and not readable by screen reader)
        safeAttributeText = replaceAll(safeAttributeText, '&', ' and ');                              // &amp; ampersand
        safeAttributeText = replaceAll(safeAttributeText, '<', ' less than ');                        // &lt;
        safeAttributeText = replaceAll(safeAttributeText, '>', ' greater than ');                     // &gt;
        safeAttributeText = replaceAll(safeAttributeText, "'", String.fromCharCode(8217));            // single quote
        safeAttributeText = replaceAll(safeAttributeText, '"', String.fromCharCode(8220));            // double quote

        // Improve readability
        safeAttributeText = replaceAll(safeAttributeText, "=", " equals ");
        safeAttributeText = replaceAll(safeAttributeText, "^", " caret ");
        safeAttributeText = replaceAll(safeAttributeText, "+", " plus ");
        // safeAttributeText = replaceAll(safeAttributeText, "-", " dash ");
        safeAttributeText = replaceAll(safeAttributeText, "~", " tilde ");
        safeAttributeText = replaceAll(safeAttributeText, "_", " underscore ");
        safeAttributeText = replaceAll(safeAttributeText, "|", " vertical bar ");
        safeAttributeText = replaceAll(safeAttributeText, "/", " slash ");

        // Add breathers around braces and brackets
        safeAttributeText = replaceAll(safeAttributeText, '{', ' { ');
        safeAttributeText = replaceAll(safeAttributeText, '}', ' } ');
        safeAttributeText = replaceAll(safeAttributeText, '(', ' ( ');
        safeAttributeText = replaceAll(safeAttributeText, ')', ' ) ');
        safeAttributeText = replaceAll(safeAttributeText, '[', ' [ ');
        safeAttributeText = replaceAll(safeAttributeText, ']', ' ] ');

        // Replace DOUBLE-SPACE with SPACE
        while (safeAttributeText.includes('  ')) {
            safeAttributeText = safeAttributeText.replaceAll('  ', ' ');
        }

        // console.log('> safeAttributeText=' + safeAttributeText);

        return safeAttributeText.trim() + ' ';
    }

    /* Function that returns a trim() version of safe aria-label text from getSafeAriaLabelText()
     * @param text : String text to be assigned to an element attribute for screen reader use (alt, aria-label attributes, etc.)
     */
    function getSafeAriaLabelTextTrim(text) {
        return getSafeAriaLabelText(text).trim();
    }

    /* Function that updates the aria-label for any input validation, based on real-time feedback as you type.
    * Generic verions of setAriaLabelForGroupAcronymInput()
    */
    function setAriaLabelForInputValidation(inputKey) {
        var $inputElem = $("#" & inputKey);
        if ($inputElem.length !== 1) { return; }

        var $inputHelpblock = $("#" & inputKey & "_helpblock");
        if ($inputHelpblock.length !== 1) { return; }

        var ariaLabelText = $inputElem.attr('data-original-aria-label');
        if (isEmpty(ariaLabelText)) {
            ariaLabelText = '';
        }

        var inputHelpblockText = $inputHelpblockText.text();
        if (!isEmpty(inputHelpblockText)) {
            ariaLabelText = inputHelpblockText + ' ' + ariaLabelText;
        }

        setTimeout(function () {
            $inputElem.attr('aria-label', getSafeAriaLabelTextTrim(ariaLabelText));
        }, 250);
    }

    /* Function that returns a screen reader audible equivalent to a number value between -999 and +999.
     * Convert numbers (integers) to text for screen reader. Numbers outside of range will be returned as a string version of the number.
     * @param n : Number between -999 and +999
     */
    function getAriaNumberName(n) {
        if (isEmpty(n) || !isNumber(n)) { return ''; }
        if (Math.abs(n) >= 1000) { return '' + n; }
        if (n < 0) { return 'minus ' + getAriaNumberName(-n); }
        if (n === 0) { return numberNames[0]; }

        var numberName = '';

        var hundreds = Math.floor(n / 100);
        if (hundreds > 0) { numberName += numberNames[hundreds] + ' hundred'; }

        n = Math.floor(n % 100);
        if (n > 0) {
            if (!isEmpty(numberName)) { numberName += ' and '; }
            if (n < 20) {
                numberName += numberNames[n];
            } else {
                numberName += tensNames[Math.floor(n / 10)];
                var ones = Math.floor(n % 10);
                if (ones > 0) {
                    numberName += ' ' + numberNames[ones];
                }
            }
        }
        return numberName.trim();
    }

    /* Function that converts event date and time information into a version suitable for the screen reader (alt text, aria-label, etc.)
     * Converts event start/end date details, as returned by the database, to a more screen reader (and removes any embedded HTML or scripts)
     * @param eventDateDetails : The event start/end date as provided by the database (including HTML for formatting inside an event carousel card or equivalent)
     */
    function convertEventDateDetailsToAriaLabelText(eventDateDetails) {
        
        var expandedTerms = {};
        // day names
        expandedTerms['Mon,'] = 'Monday,';
        expandedTerms['Tue,'] = 'Tuesday,';
        expandedTerms['Wed,'] = 'Wednesday,';
        expandedTerms['Thu,'] = 'Thursday,';
        expandedTerms['Fri,'] = 'Friday,';
        expandedTerms['Sat,'] = 'Saturday,';
        expandedTerms['Sun,'] = 'Sunday,';
        // month names
        expandedTerms[' Jan '] = ' January ';
        expandedTerms[' Feb '] = ' February ';
        expandedTerms[' Mar '] = ' March ';
        expandedTerms[' Apr '] = ' April ';
        expandedTerms[' May '] = ' May ';
        expandedTerms[' Jun '] = ' June ';
        expandedTerms[' Jul '] = ' July ';
        expandedTerms[' Aug '] = ' August ';
        expandedTerms[' Sep '] = ' September ';
        expandedTerms[' Oct '] = ' October ';
        expandedTerms[' Nov '] = ' November ';
        expandedTerms[' Dec '] = ' December ';
        // other
        expandedTerms['\n'] = ' ';
        expandedTerms[' .'] = '.';

        eventDateDetails = getSafeAriaLabelText(eventDateDetails);
        for (var k in expandedTerms) {
            eventDateDetails = replaceAll(eventDateDetails, k, expandedTerms[k]);
        }
        return eventDateDetails.trim();
    }

    /* Function that adds punctuation to the end of a block of text, typically to prepare multiple blocks of text for the screen reader.
     * If punctuationChar is not empty AND the text is already terminated with a punctuation character, the existing punctuation character will be replaced with the value specified by 'punctuationChar'.
     * Used to prepar ALT text or aria-label, etc.
     * @param text             : String text to be terminated with punctuation.
     * @param punctuationChar  : The desired punctuation to be placed at the end of 'text'. If empty, defaults to a period '.', when provided, will replace any existing punctuation at the end of 'text'.
     */
    function addAriaLabelPunctuation(text, punctuationChar) {
        var tailPadding = ' ';
        if (isEmpty(text)) { return tailPadding; }

        text = text.trim();
        var hasPunctuation = text.endsWith('.') || text.endsWith('!') || text.endsWith('?') || text.endsWith(':') || text.endsWith(';') || text.endsWith('-') || text.endsWith(',');

        if (isEmpty(punctuationChar)) {            
            // Don't force the punctuation - just use a '.' if there is no punctuation present
            if (!hasPunctuation) { text = text + '.'; }
            return text + tailPadding;
        } else if (text.endsWith(punctuationChar)) {
            // already ends with the specified punctuation
            return text + tailPadding;
        } else if (hasPunctuation) {
            // If text length is 1, we have a single-character punctuation mark to begin with - equivalent to an empty string
            if (text.length <= 1) { return tailPadding; }

            // Remove existing punctuation and replace with specified punctuationChar
            return text.substring(0, text.length - 1) + punctuationChar + tailPadding;
        }
        return text + punctuationChar + tailPadding;
    }

    // END refactoring 2021-01-06

    /* --------------------------------------------------------------------------------------------------
     * Get the UI label for an input and format for use in an aria-label or alt attribute (or equivalent)
     * - We only want the first sentence (or text up to a period) if the label has a lot of text
     * - We also want to remove the word "Select" from the start of the label if present
     * @param {string} textContent : The visible UI text label for an input
     */
    function extractVisibleLabel(textContent) {
        if (isEmpty(textContent)) {
            console.warn('textContent is empty');
            return '';
        }

        var ex = textContent.indexOf('.');
        if (ex > 0)
        {
            textContent = textContent.substring(0, ex);
        }
        textContent = textContent.replace("Select ", "");

        return textContent.trim();
    }

    /* --------------------------------------------------------------------------------------------------
     * Attempt to determine the visible "UI" label for an element from the element's aria-label attribute.
     * - May be modified/shortened if the aria-label is long or represents a SELECT dropdown input
     * @param {Element} elem  : An input element
     */
    function extractVisibleLabelFromAriaLabel(elem) {

        if (isEmpty(elem)) {
            console.error('elem is empty.');
            return '';
        }

        var ariaLabel = elem.getAttribute('aria-label');
        if (isEmpty(ariaLabel)) {
            return '';
        }

        return extractVisibleLabel(ariaLabel);
    }

    /* --------------------------------------------------------------------------------------------------
     * Extract the visible UI label from an input's associated <label> element's text content
     * - May be modified/shortened if the label text is long or represents a SELECT dropdown input
     * @param {Element} labelElem  : Label element <label> associated with an input
     */
    function extractVisibleLabelFromLabelTextContent(labelElem) {

        if (isEmpty(labelElem)) {
            console.error('labelElem is empty.');
            return '';
        }

        if (isEmpty(labelElem.innerHTML)) {
            return '';
        }

        //
        // we don't want to use the textContent of the label if it includes any functional code
        // i.e. anything other than pure label content and associated standard formatting
        // <span> is OK for formatting for example
        //
        if (labelElem.innerHTML.includes('<script') || labelElem.innerHTML.includes('<style') || labelElem.innerHTML.includes('$(')) {
            console.error('> extractVisibleLabelFromLabelTextContent() unusable content.');
            return '';      
        }

        return extractVisibleLabel(labelElem.textContent);
    }

    /* --------------------------------------------------------------------------------------------------
     * Extract the visible UI label from an input's associated <label> element
     * - First check if the label has an aria-label attribute defined
     * - If not, try to use the label's text content
     * 
     * - May be modified/shortened if the label text is long or represents a SELECT dropdown input
     * 
     * @param {Element} labelElem  : Label element <label> associated with an input
     * 
     */
    function extractVisibleLabelFromLabel(labelElem)
    {
        if (isEmpty(labelElem)) {
            console.error('labelElem is empty.');
            return '';
        }

        var labelText = extractVisibleLabelFromAriaLabel(labelElem);
        if (isEmpty(labelText)) {
            labelText = extractVisibleLabelFromLabelTextContent(labelElem);
            if (isEmpty(labelText)) {
                labelText = '';
            }
        }

        labelText = labelText.trim().replace('<', '&lt;');

        return labelText;
    }

    /* --------------------------------------------------------------------------------------------------
     * Get the <label> element associated with an input (or equivalent)
     * - The CG coding convention appears to use 'label-for' + inputId as the ID for the associated <label>
     * - can be expanded if necessary if there are more patterns to consider
     * 
     * @param {string} elemId  : The ID of an input (or equivalent) for which we want to find the associated label element
     * 
     */
    function getLabelForElement(elemId)
    {
        // @REMOVE-FROM here
        if (isEmpty(elemId)) {
            console.error('elemId is empty');
            return;
        }
        // @REMOVE-TO here

        var elemLabelId = 'label-for-' + elemId;
        return document.getElementById(elemLabelId);
    }

    /* --------------------------------------------------------------------------------------------------
     * Get the UI label text for an element (typically an input element).
     * - First check if the element has an associated label (and if so, the text from the <label>)
     * - If we don't find anything, use the input's aria-label
     * 
     * The returned string will be shortened if the label text is long or if it starts with "Select " (we'll remove the "Select " prefix)
     * 
     * @param {Element} elem  : The input or equivalent element for which we want to get the UI visible label text
     * 
     */
    function getLabelTextForElement(elem) {
        if (isEmpty(elem)) {
            console.error('elem is empty');
            return '';
        }

        var labelText = '';
        if (!isEmpty(elem.id)) {
            var labelElem = getLabelForElement(elem.id);
            if (!isEmpty(labelElem)) {
                labelText = extractVisibleLabelFromLabel(labelElem);
            }
        }

        if (isEmpty(labelText)) {
            labelText = extractVisibleLabelFromAriaLabel(elem);
        }

        return labelText;
    }

    /* --------------------------------------------------------------------------------------------------
     * Get the UI label text for an element (typically an input element)
     * - First check if the element has an associated label (and if so, the text from the <label>)
     * - If we don't find anything, use the input's aria-label
     * - If the label text is prefixed with an asterix (a required input) we'll remove this as well
     * 
     * The returned string will be shortened if the label text is long or if it starts with "Select " (we'll remove the "Select " prefix)
     * 
     * @param {Element} elem  : The input or equivalent element for which we want to get the UI visible label text
     * 
     */
    function getLabelTextForElementWithoutAsterix(elem) {
        var labelText = getLabelTextForElement(elem);
        if (isEmpty(labelText)) {
            labelText = '';
        } else {
            if (labelText.startsWith('*')) {
                labelText = labelText.substring(1).trim();
            }
        }
        return labelText;
    }

    /* --------------------------------------------------------------------------------------------------
     * ADD aria-label attribute to SELECT options that include the up-arrow or down-arrow representing ordering (ascending or descending)
     * - For list pages, if the filter bar includes an "Ordering" dropdown, we want to improve the screen reader content so it specifically indicates if the order is ascending or descending
     * 
     */
    function addAriaLabelsToOrderingSelectOptions() {
        setTimeout(function () {
            var selectOptions = $('select#select_order option');
            if (selectOptions && selectOptions.length) {
                for (var index=0; index<selectOptions.length; index++) {
                    var option = selectOptions.get(index);
                    var optionText = option.textContent.trim();
                    var ariaLabel = null;
                    if (optionText.endsWith('▲')) {
                        ariaLabel = optionText.substring(0, optionText.length - 1).trim() + ', Ascending A to Z';
                    } else if (optionText.endsWith('▼')) {
                        ariaLabel = optionText.substring(0, optionText.length - 1).trim() + ', Descending Z to A';
                    }
                    if (ariaLabel !== null) {
                        option.setAttribute('aria-label', ariaLabel);
                    }
                }
            }
        }, 100);
    }

    /* --------------------------------------------------------------------------------------------------
     * Function that updates screen reader context for expandable sidebar menus that automatically
     * load one of the sub-menu pages when expanded. Changing page context when expanding a menu
     * is unexpected behavior for keyboard users and so we'll add additional context notifying
     * users that the page context will change when they expand the menu.
     * 
     * Not all expandable menus change page context, we're only targeting the pages that auto-load
     * a sub-menu page (all call a transition nav or save recently function in the link onclick).
     * 
     */
    function addAriaLabelToExpandableSidebarLinks() {
        setTimeout(function () {
            try {
                // The Sidebar expandable menus that load a sub-menu page when expanded all have an onclick defined
                // we're not interested in expandable menus without onclick as they don't automatically load a sub-menu
                let onclickCategoryTags = [];
                onclickCategoryTags.push('transitionAdminNav('); // Topbar > Admin
                onclickCategoryTags.push('saveAdminRecently('); // Topbar > Admin
                onclickCategoryTags.push('transitionAccountNav('); // Topbar > Account
                onclickCategoryTags.push('transitionAppNav('); // Topbar > Home
                onclickCategoryTags.push('transitionGroupsNav('); // Home > Feed
                onclickCategoryTags.push('transitionManageNav('); // Manage Group

                let expandableSidebarLinks = $('div#sidebar-menu ul#side-menu > li > a[aria-expanded]');
                for (var i=0; i<expandableSidebarLinks.length; i++) {
                    let expandableSidebarLink = expandableSidebarLinks[i];
                    let onclick = $(expandableSidebarLink).attr('onclick');
                    if (onclick) {
                        let onclickTag = null;
                        for (var oct=0; oct<onclickCategoryTags.length; oct++) {
                            let onclickCategoryTag = onclickCategoryTags[oct];
                            if (onclick.includes(onclickCategoryTag)) {
                                let sx = onclick.indexOf(onclickCategoryTag);
                                let ex = onclick.indexOf(',', sx);
                                if (ex > sx) {
                                    if (onclickCategoryTag === 'saveAdminRecently(') {
                                        sx = ex + 1;
                                        ex = onclick.indexOf(',', sx);
                                        if (ex > sx) {
                                            onclickTag = onclick.substring(sx, ex).trim();
                                        }
                                    } else {
                                        onclickTag = onclick.substring(sx, ex).trim();
                                        if (onclickCategoryTag === 'transitionAdminNav(') {
                                            onclickTag = onclickTag.replace('transitionAdminNav(', '');                                    
                                        }
                                    }
                                }
                            }    
                        }
                        if (onclickTag) {
                            let j = 0;
                            let defaultOpenSubMenuName = null;
                            let subMenuLink = null;
                            let subMenuLinks = $(expandableSidebarLink).parent().find('ul > li > a');
                            while (j<subMenuLinks.length && defaultOpenSubMenuName === null) {
                                subMenuLink = subMenuLinks[j];
                                subMenuLinkOnclick = $(subMenuLink).attr('onclick');
                                if (subMenuLinkOnclick && subMenuLinkOnclick.includes(onclickTag)) {
                                    defaultOpenSubMenuName = $(subMenuLink).attr('aria-label');
                                    if (defaultOpenSubMenuName.indexOf('.') > 0) {
                                        defaultOpenSubMenuName = defaultOpenSubMenuName.substring(0, defaultOpenSubMenuName.indexOf('.')).trim();
                                    }
                                }
                                j++;
                            }
                            defaultOpenSubMenuName = (defaultOpenSubMenuName) ? defaultOpenSubMenuName : 'default sub-menu';
                            if ($(expandableSidebarLink).attr('acc-sidebar-config') !== 'true') {
                                let expandableSidebarLinkAriaDescription = $(expandableSidebarLink).attr('aria-description');
                                if (isEmpty(expandableSidebarLinkAriaDescription)) {
                                    expandableSidebarLinkAriaDescription = '';
                                }
                                expandableSidebarLinkAriaDescription = addAriaLabelPunctuation(expandableSidebarLinkAriaDescription, '.');
                                expandableSidebarLinkAriaDescription = expandableSidebarLinkAriaDescription.trim();
                                expandableSidebarLinkAriaDescription += ' Automatically loads the ' + defaultOpenSubMenuName + ' page when expanded.';
                                $(expandableSidebarLink).attr('aria-description', expandableSidebarLinkAriaDescription.trim());
                                $(expandableSidebarLink).attr('acc-sidebar-config', 'true');
                            }
                        }
                    }
                }
            } catch (e) {
                console.error(e);
            }
        }, 667); // wait for the sidebar to render
    }

/*-----------------------------------------------------*\
 * @SELECTIZE
 * 
 * Setup and manage keyboard navigation for SELECT inputs
 * (multi-select checkbox list item dropdowns)
 * 
\*-----------------------------------------------------*/

    /* --------------------------------------------------------------------------------------------------
     * SELECTIZE management for dropdown selects that "Start typing and wait for suggestions" such as user selects with real-time feedback on partial text
     * - help with aria-label on the base input when typing and navigating result list
     * - for example, after typing 3+ characters, if the result list has results we update the aria-label to read the length of the result list
     * - this is just the setup to manage the dropdown/select
     * - set keyup event on up/down arrows so the input's aria-label is updated to read the current selection as we navigate up/down the list
     * 
     * @param {string}  wrapperId                 : The container in which the <input> associated with the select dropdown resides
     * @param {string}  selectizeInputAriaLabel   : The aria-label associated with the select (to be assigned to the UI select)
     * @param {boolean} isSelectizeInputRequired  : True if this is a required input (defaults to false if empty)
     * 
     */
    var selectizeResultCount = {};  // count of the actual number of list items added (html) to the result list during build; incremented during loop/build
    var selectizeResultLength = {}; // the length of the result list used to build the (html) results; set once when we receive the results
    function setupSelectizeInput(wrapperId, selectizeInputAriaLabel, isSelectizeInputRequired) {
        // @REMOVE-FROM here
        console.log('> setupSelectizeInput() begins...');

        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        // @REMOVE-TO here

        if (isEmpty(selectizeInputAriaLabel)) {
            selectizeInputAriaLabel = '';
        }
        if (isEmpty(isSelectizeInputRequired)) {
            isSelectizeInputRequired = false;
        }

        var selectizeInputId = '#' + wrapperId + ' input';
        var selectizeInput = $(selectizeInputId);
        if (isEmpty(selectizeInput)) {
            console.error('selectizeInput is empty');
            return;
        }

        var dataOriginalAriaLabel = addAriaLabelPunctuation(selectizeInputAriaLabel);
        if (isSelectizeInputRequired === true) {
            dataOriginalAriaLabel += ' Required.';
        }
        dataOriginalAriaLabel = getSafeAriaLabelTextTrim(dataOriginalAriaLabel);

        selectizeInputAriaLabel = dataOriginalAriaLabel;
        selectizeInputAriaLabel += ' Start typing and wait for suggestions.';

        console.log('> setupSelectizeInput() selectizeInputAriaLabel=' + selectizeInputAriaLabel);
        selectizeInput.attr('data-original-aria-label', dataOriginalAriaLabel);
        selectizeInput.attr('aria-label', selectizeInputAriaLabel);
        selectizeInput.attr('tabindex', '0');

        selectizeInput.keyup(function (event) {
            console.log('> KEYUP event.which=' + event.which);
            if (event.which == 38 || event.which == 40) {
                // up/down arrows
                setAriaLabelForSelectize(wrapperId);
            }
        });

        console.log('> wrapperId=' + wrapperId + ', selectizeInputAriaLabel=' + selectizeInputAriaLabel);
        console.log('> setupSelectizeInput() wrapperId=' + wrapperId + ', complete!');
    }

    /* --------------------------------------------------------------------------------------------------
     * Clear the result count and length for the specified select input.
     * 
     * @param {string}  wrapperId  : The container in which the <input> associated with the select dropdown resides
     * 
     */
    function clearSelectizeResults(wrapperId) {
        // @REMOVE-FROM here
        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        // @REMOVE-TO here
        // console.log('> clearSelectizeResults() wrapperId=' + wrapperId);
        if (!isEmpty(selectizeResultCount[wrapperId])) {
            delete selectizeResultCount[wrapperId];
        }
        if (!isEmpty(selectizeResultLength[wrapperId])) {
            delete selectizeResultLength[wrapperId];
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns the length of the result list for the given select input.
     * 
     * @param {string}  wrapperId  : The container in which the <input> associated with the select dropdown resides
     * 
     */
    function getSelectizeResultLength(wrapperId) {
        // @REMOVE-FROM here
        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        // @REMOVE-TO here
        if (isEmpty(selectizeResultLength[wrapperId])) {
            return 0;
        }
        // console.log('> getSelectizeResultLength() wrapperId=' + wrapperId + ', selectizeResultLength[wrapperId]=' + selectizeResultLength[wrapperId]);
        return selectizeResultLength[wrapperId];
    }

    /* --------------------------------------------------------------------------------------------------
     * Sets the length of the result list for the given select input (based on result from ajax call).
     * 
     * @param {string}  wrapperId  : The container in which the <input> associated with the select dropdown resides
     * 
     */
    function setSelectizeResultLength(wrapperId, count) {
        // @REMOVE-FROM here
        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        if (isEmpty(count)) {
            console.error('count is empty');
            return;
        }
        // @REMOVE-TO here
        selectizeResultLength[wrapperId] = count;
        // console.log('> setSelectizeResultLength() wrapperId=' + wrapperId + ', count=' + count);
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns the number of list items currently in the visible UI select dropdown associated with the input/select.
     * - This is different from the ajax request result length as ajax results appear to be added to the local list items, but previous query list items also still exist (and are often active/visible)
     * 
     * @param {string}  wrapperId  : The container in which the <input> associated with the select dropdown resides
     * 
     */
    function getSelectizeResultCount(wrapperId) {
        var listItemsSelector = '#' + wrapperId + ' div.selectize-dropdown-content div';
        var listItems = $(listItemsSelector);
        return (listItems === undefined || listItems === null) ? 0 : listItems.length / 2;
    }

    /* --------------------------------------------------------------------------------------------------
     * Increment the count of the number of list items added to the select dropdown referenced by wrapperId.
     * 
     * @param {string}  wrapperId  : The container in which the <input> associated with the select dropdown resides
     * 
     */
    function incrementAndGetSelectizeResultCount(wrapperId) {   
        // @REMOVE-FROM here
        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        // @REMOVE-TO here
        if (isEmpty(selectizeResultCount[wrapperId])) {
            selectizeResultCount[wrapperId] = 0;
        }
        selectizeResultCount[wrapperId] = selectizeResultCount[wrapperId] + 1;
        return selectizeResultCount[wrapperId];
    }

    /* --------------------------------------------------------------------------------------------------
     * For multi-select inputs with checkbox list items, set the link's role (to checkbox) and if the checkbox is active, set the 'checked' attribute as well (or clear otherwise)
     * 
     * @param {string} selectId  : ID of the select dropdown
     * 
     */
    function setOptionLinkRolesForSelectMultipleNewWithClass(selectId) {

        if (selectId.startsWith('#')) {
            selectId = selectId.substring(1);
        }

        var optionLinks = $('#' + selectId).next().find('ul li a');
        for (var i=0; i<optionLinks.length; i++) {
            var optionLink = $(optionLinks.get(i));
            $(optionLink).attr('role', 'checkbox');

            if ($(optionLink).parent().hasClass('active')) {
                $(optionLink).attr('checked', '');
                $(optionLink).find('div input').attr('checked', '');
            } else {
                $(optionLink).removeAttr('checked');
                $(optionLink).find('div input').removeAttr('checked');
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Keyboard input manager for dropdown/select inputs (BuildSelect using checkbox options inside select)
     * - Determine the character associated with keyup events (regardless of laptop/desktop keyboard or Windows/Mac OS)
     * - Used in the "skip to nearest item" functionality for dropdown selects allowing "type to find the nearest select item"
     * 
     * @param {number}  which       : KeyboardEvent.which value (from associated keyup event)
     * @param {boolean} isShiftKey  : True if the shift key was pressed
     * 
     */
    function getCharFromKeyup(which, isShiftKey) {
        // @REMOVE-FROM here
        if (isEmpty(which)) {
            console.error('which is empty');
            return;
        } else if (!isNumber(which)) {
            console.error('which is not a number');
            return;
        }

        if (isEmpty(isShiftKey)) {
            console.error('isShiftKey is empty');
            return;
        } else if (!isBoolean(isShiftKey)) {
            console.error('isShiftKey is not a boolean');
            return;
        }
        // @REMOVE-TO here

        var block186LowerCase = ';=,-./`';
        var block186UpperCase = ':+<_>?~';
        var block219LowerCase = "[\\]'";
        var block219UpperCase = '{|}"';
        var lettersLowerCase = 'abcdefghijklmnopqrstuvwxyz';
        var lettersUpperCase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
        var numbersLowerCase = '0123456789';
        var numbersUpperCase = ')!@#$%^&*(';
        var numpadChars = '*+ -./'

        console.log('> getCharFromKeyup() isShiftKey=' + isShiftKey + ', which=' + which);

        var charValue = null;
        if (isShiftKey) {
            if (which >= 48 && which <= 57) {
                charValue = numbersUpperCase[which - 48];
            } else if (which >= 65 && which <= 90) {
                charValue = lettersUpperCase[which - 65];
            } else if (which >= 96 && which <= 105) {
                charValue = numbersLowerCase[which - 96];
            } else if (which >= 106 && which <= 111) {
                charValue = numpadChars[which - 106];
            } else if (which >= 186 && which <= 192) {
                charValue = block186UpperCase[which - 186];
            } else if (which >= 219 && which <= 222) {
                charValue = block219UpperCase[which - 219];
            }
        } else {
            if (which >= 48 && which <= 57) {
                charValue = numbersLowerCase[which - 48];
            } else if (which >= 65 && which <= 90) {
                charValue = lettersLowerCase[which - 65];
            } else if (which >= 96 && which <= 105) {
                charValue = numbersLowerCase[which - 96];
            } else if (which >= 106 && which <= 111) {
                charValue = numpadChars[which - 106];
            } else if (which >= 186 && which <= 192) {
                charValue = block186LowerCase[which - 186];
            } else if (which >= 219 && which <= 222) {
                charValue = block219LowerCase[which - 219];
            }
        }

        console.log('> charValue=' + charValue);

        if (isEmpty(charValue)) {
            charValue = null;
        }

        return charValue;
    }

    /* --------------------------------------------------------------------------------------------------
     * Returns a number indicating the sort order of the select dropdown list.
     * - For dropdown multi-selects that support "type to skip" to the nearest list item
     * 
     * -1 = Descending (Z-A)
     *  0 = Not sorted or unordered
     * +1 = Ascending (A-Z)
     * 
     */
    function getDropdownSortOrder(selectId) {
        if (selectId.startsWith('#')) {
            selectId = selectId.substring(1);
        }

        var optionLabels = $('#' + selectId).next().find('ul li a div label');
        var prevTitle = null;
        var isAscending = true;
        var isDescending = true;
        for (var i=0; i<optionLabels.length; i++) {
            var optionLabel = $(optionLabels.get(i));
            var title = $(optionLabel).attr('title').trim().toLowerCase();
            if (prevTitle === null) {
                prevTitle = title;
            } else {
                if (title.localeCompare(prevTitle) >= 0) {
                    isDescending = false;
                } else if (title.localeCompare(prevTitle) <= 0) {
                    isAscending = false;
                }
            }
        }

        if (isAscending) {
            return +1;
        } else if (isDescending) {
            return -1;
        }

        return 0;
    }

    /* --------------------------------------------------------------------------------------------------
     * Set keyup event handler on select facilitating type-to-skip to the nearest record based on what the user types
     * 
     * - Start typing, focus will skip to the nearest list item matching what you've typed
     * - Press Backspace/Delete to clear the typing buffer and start over/try again
     * - Press ESCAPE to collapse the dropdown (close the select)
     * - Press ENTER or SPACE to check a list item (check/uncheck depending on state)
     * 
     * @param {string} selectId     : ID of the associated select dropdown
     * @param {number} optionCount  : The number of items in the select dropdown list.
     * 
     * NOTE optionCount is no longer used (and useApplicationMode is set to true by default). This should be cleaned up in next round of refactoring.
     * 
     */
    function manageKeyboardForSelectMultipleNewWithClass(selectId, optionCount) {

        if (selectId.startsWith('#')) {
            selectId = selectId.substring(1);
        }

        if (isEmpty(optionCount)) {
            optionCount = 0;
        }

        // @INVESTIGATE rework the code below (remove useApplicationMode boolean; it's always true)
        var useApplicationMode = false;
        if (optionCount >= 0) {
            // OCT-15 use application mode perpetually for JAWS, NVDA and VoicdOver support
            useApplicationMode = true;
        }

        var chars = '';
        var sortOrder = 0; // -1 descending : +1 ascending : 0 not sorted

        setOptionLinkRolesForSelectMultipleNewWithClass(selectId);

        if (useApplicationMode) {
            // flag this area as an application to prevent NVDA/etc. from handling keyboard input
            $('#' + selectId).parent().attr('role', 'application');

            setTimeout(function () {
                sortOrder = getDropdownSortOrder(selectId);
                // console.log('> sortOrder=' + sortOrder);
            }, 100);
        }

        var selectButton = $('#' + selectId).next().find('button');
        if (!isEmpty(selectButton) && selectButton.length === 1) {
            var originalAriaLabel = $(selectButton).attr('aria-label');
            $(selectButton).attr('data-original-aria-label', originalAriaLabel);

            if (useApplicationMode) {
                var newAriaLabel = addAriaLabelPunctuation(originalAriaLabel).trim() + ' Enter or arrow down to expand the list. Start typing to navigate to the nearest option. Backspace or delete to empty typing buffer.'
                $(selectButton).attr('aria-label', newAriaLabel);
            }
        }

        $('#' + selectId).next().keyup(function (event) {
            console.log('> SELECT keyup selectId=' + selectId + ', sortOrder=' + sortOrder);

            if (isAccessibilityJSVerbose) {
                console.log('keyup');
            }
            if (event.which === 27) {
                // escape key
                if (isAccessibilityJSVerbose) {
                    console.log('> keyup event.which=' + event.which + ' *** ESC escape key pressed!');
                }
                $('#' + selectId).next().removeClass('open');
                selectButton.trigger("focus")
            }
            
            if (useApplicationMode) {
                if (event.which === 8 || event.which === 46) {
                    // backspace and delete
                    if (isAccessibilityJSVerbose) {
                        console.log('> keyup event.which=' + event.which);
                    }
                    chars = '';
                } else if (event.which === 13 || event.which === 32) {
                    // enter or space bar
                    var activeElem = document.activeElement;
                    setActiveOptionAriaLabelForSelectMultipleNewWithClass(activeElem);
                } else {
                    // all other keys
                    if (isAccessibilityJSVerbose) {
                        console.log('> keyup event.which=' + event.which);
                    }

                    var char = getCharFromKeyup(event.which, event.shiftKey);
                    if (!isEmpty(char)) {
                        char = char.toLowerCase();
                        console.log('> char=' + char);
                        chars += char;
                        console.log('> chars=' + chars);

                        var hasFocused = false;
                        var optionLabels = $('#' + selectId).next().find('ul li a div label');
                        console.log('> optionLabels.length=' + optionLabels.length);
                        for (var i=0; i<optionLabels.length; i++) {
                            var optionLabel = $(optionLabels.get(i));
                            var title = $(optionLabel).attr('title');
                            if (!hasFocused && !isEmpty(title)) {
                                title = title.trim().toLowerCase();
                                console.log('> title=' + title);
                                if (!isEmpty(chars)) {
                                    var takeFocus = false;
                                    if (title.startsWith(chars)) {
                                        console.log('> *** FOCUS{a} on=' + title);
                                        takeFocus = true;
                                    } else {
                                        if (sortOrder > 0 && title.localeCompare(chars) > 0) {
                                            console.log('> *** FOCUS{ascending} on=' + title);
                                            takeFocus = true;
                                        } else if (sortOrder < 0 && title.localeCompare(chars) < 0) {
                                            console.log('> *** FOCUS{descending} on' + title);
                                            takeFocus = true;
                                        }
                                    }
                                }

                                if (takeFocus) {
                                    hasFocused = true;
                                    var option = $(optionLabel).parent().parent();
                                    setActiveOptionAriaLabelForSelectMultipleNewWithClass(option);
                                    option.trigger("focus");
                                    break;
                                }
                            }
                        }
                    }
                }
            }
        });
    }

    /* --------------------------------------------------------------------------------------------------
     * Set the aria-label on the input associated with a dropdown "Type to search" select
     * - As the user types, the results may change (add new results, reduce the result set)
     * - Using up/down arrow, as the user navigates the results we update the input's aria-label to match the currently selected list item
     * 
     * @param {string} wrapperId          : The container in which the <input> associated with the select dropdown resides
     * @elem {string}  inputLabelText     : The visible UI label text for the input/select.
     * @elem {boolean} includeResultCount : If true, we'll include "7 Results." or equivalent ("N Results." depending on the number of items in the select dropdown)
     * 
     */
    function setAriaLabelForSelectize(wrapperId, inputLabelText, includeResultCount) {
        // @REMOVE-FROM here
        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return;
        }
        // @REMOVE-TO here
        if (isEmpty(inputLabelText)) {
            inputLabelText = '';
        }
        if (isEmpty(includeResultCount)) {
            includeResultCount = false;
        }
        console.log('> setAriaLabelForSelectize() wrapperId=' + wrapperId);
        var activeItemSelector = '#' + wrapperId + ' div.selectize-dropdown-content div.active';
        // activeItemSelector = 'div.selectize-dropdown-content div.active'
        var activeItem = $(activeItemSelector);
        if (activeItem !== undefined && activeItem !== null && activeItem.length == 1) {
            var dataValue = activeItem.attr('data-value');
            var listItemAriaLabel = activeItem.attr('aria-label');
            console.log('> KEYUP activeItem.dataValue=' + dataValue + ', activeItem.aria-label=' + listItemAriaLabel);
            if (!isEmpty(listItemAriaLabel)) {
                var originalInputAriaLabel = $('#' + wrapperId + ' input').attr('data-original-aria-label');
                if (isEmpty(originalInputAriaLabel)) {
                    console.error('> NOTE setAriaLabelForSelectize() originalInputAriaLabel is empty');
                    originalInputAriaLabel = inputLabelText;
                }
                var inputAriaLabel = '';
                if (includeResultCount === true) {
                    inputAriaLabel = inputAriaLabel + getSelectizeResultCount(wrapperId) + ' Results. ';
                }
                inputAriaLabel = inputAriaLabel + ' ' + listItemAriaLabel + ' ';
                if (!isEmpty(originalInputAriaLabel)) {
                    inputAriaLabel = inputAriaLabel + addAriaLabelPunctuation(originalInputAriaLabel);
                }
                inputAriaLabel = inputAriaLabel + ' Use up and down arrows to navigate list results.';
                inputAriaLabel = getSafeAriaLabelTextTrim(inputAriaLabel);
                console.log('> KEYUP inputAriaLabel=' + inputAriaLabel);
                $('#' + wrapperId + ' input').attr('aria-label', inputAriaLabel);
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Set the aria-label on the input associated with a dropdown "Type to search" select
     * - Depending on the potential length of the result set (or when querys are known to take more time than usual)
     * - we will wait the specified number of millis before calling setAriaLabelForSelectize()
     * 
     * @param {number} waitMillis         : Number of milliseconds to wait before updating the aria-label on the <input> associated with the select
     * @param {string} wrapperId          : The container in which the <input> associated with the select dropdown resides
     * @elem {string}  inputLabelText     : The visible UI label text for the input/select.
     * @elem {boolean} includeResultCount : If true, we'll include "7 Results." or equivalent ("N Results." depending on the number of items in the select dropdown)

     */
    function setAriaLabelForSelectizeDelayed(waitMillis, wrapperId, inputLabelText, includeResultCount) {
        // @REMOVE-FROM here
        if (isEmpty(waitMillis)) {
            console.error('waitMillis is empty');
            return;
        }
        // @REMOVE-TO here
        setTimeout(function () {
            setAriaLabelForSelectize(wrapperId, inputLabelText, includeResultCount);
        }, waitMillis);
    }

    /* --------------------------------------------------------------------------------------------------
     * Build and return an aria-label for list-item results representing students.
     * 
     * @param {string} wrapperId     : The container in which the <input> associated with the select dropdown resides
     * @param {string} full_name     : Student's full name
     * @param {string} email         : Student's email address (may be empty)
     * @param {string} student_type  : Student type (may be empty)
     * @param {string} student_yog   : Student year of graduation (may be empty)
     * 
     */
    function getAriaLabelForSelectizeStudents(wrapperId, full_name, email, student_type, student_yog) {
        // @REMOVE-FROM here
        console.log('> getAriaLabelForSelectizeStudents() begins...');

        if (isEmpty(wrapperId)) {
            console.error('wrapperId is empty');
            return '';
        }
        // @REMOVE-TO here

        if (isEmpty(full_name)) {
            console.error('full_name is empty');
            return '';
        }

        var resultCount = incrementAndGetSelectizeResultCount(wrapperId);
        var resultsLength = getSelectizeResultLength(wrapperId);
        if (resultsLength < 1) {
            console.log('> NOTE getAriaLabelForSelectizeStudents() resultsLength < 1');
        }

        var listItemAriaLabel = '';
        // listItemAriaLabel = listItemAriaLabel + 'Option ' + resultCount + ' of ' + resultsLength + '. ';

        listItemAriaLabel += getSafeAriaLabelTextTrim(full_name);
        if (!isEmpty(email)) {
            listItemAriaLabel += ', email ' + getSafeAriaLabelTextTrim(email);
        }
        var studentDetails = '';
        if (!isEmpty(student_type)) {
            studentDetails += getSafeAriaLabelTextTrim(student_type) + ' ';
        }
        if (!isEmpty(student_yog)) {
            studentDetails += 'graduating ' + getSafeAriaLabelTextTrim(student_yog);
        }
        if (!isEmpty(studentDetails)) {
            listItemAriaLabel += ', ' + getSafeAriaLabelTextTrim(studentDetails);
        }
        listItemAriaLabel = addAriaLabelPunctuation(listItemAriaLabel);

        return getSafeAriaLabelTextTrim(listItemAriaLabel);
    }

    /* --------------------------------------------------------------------------------------------------
     * Set the aria-label for the active list option (including checked/unchecked state).
     * - Can be improved by moving code closer to the ajax results and/or onClick function that handles check/uncheck for the list items
     * - Discuss with Adrien
     * 
     * @param (Element) aOption  : The <a> link of the active result list item
     * 
     */
    function setActiveOptionAriaLabelForSelectMultipleNewWithClass(aOption) {
        var aOptionParent = $(aOption).parent();
        if ($(aOptionParent).is('li')) {
            var isChecked = false;
            if (aOptionParent.hasClass('active')) {
                isChecked = true;
                if (isAccessibilityJSVerbose) {
                    console.log('> aOptionParent is ACTIVE');
                }
            }
            var aOptionLabel = $(aOption).find('div label');
            if (!isEmpty(aOptionLabel) && aOptionLabel.length === 1) {
                var labelText = aOptionLabel.text();
                var labelAriaLabel = addAriaLabelPunctuation(labelText);
                if (isChecked) {
                    labelAriaLabel += ' Checked.'
                }
                labelAriaLabel = getSafeAriaLabelTextTrim(labelAriaLabel);
                if (isAccessibilityJSVerbose) {
                    console.log('> aOptionentLabel=' + labelText);
                    console.log('> labelAriaLabel=' + labelAriaLabel);
                }
                $(aOptionLabel).attr('aria-label', labelAriaLabel);
                $(aOption).attr('aria-label', labelAriaLabel);
                if (isChecked) {
                    $(aOption).attr('checked', '');
                } else {
                    $(aOption).removeAttr('checked');
                }
            }
            var aOptionInput = $(aOption).find('div input');
            if (!isEmpty(aOptionInput) && aOptionInput.length === 1) {
                if (isChecked) {
                    $(aOptionInput).attr('checked', '');
                } else {
                    $(aOptionInput).removeAttr('checked');
                }
            }
        }
    }

/*-----------------------------------------------------*\
 * @SLIDESHOW-CAROUSEL
 * 
 * Setup and manage keyboard navigation for event carousels
 * (event carousel implemented using slick slideshow)
 * 
\*-----------------------------------------------------*/

    /* Function that reduces the slick slideshow slidesToScroll value to 1 when in acc-keyboard mode
     * (called automatically when acc-keyboard-mode is enabled; otherwise no action).
     */
    function setupSlickAccKeyboardMode() {
        let slideshows = $('.slick-initialized');
        for (var i=0; i<slideshows.length; i++) {
            if ($(slideshows[i]).attr('data-cg-slick') !== 'true') {
                $(slideshows[i]).attr('data-cg-slick', 'true');
                $(slideshows[i]).slick('slickSetOption', 'slidesToScroll', 1);
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Setup keyboard navigation management on the website slideshow.
     * - Add keyup, focus and blur events to the slideshow controls
     * 
     * @param {string} divWrapperId             : ID of the slideshow container div
     * @param {string} h2SlideshowPreviousId    : ID of the slideshow previous button
     * @param {string} h2SlideshowNextId        : ID of the slideshow next button
     * @param {string} spanSlideshowPreviousId  : ID of the span inside the slideshow previous button
     * @param {string} spanSlideshowNextId      : ID of the span inside the slideshow next button
     * 
     * Suspect we can improve by removing parameters [spanSlideshowPreviousId,spanSlideshowNextId]
     * 
     */
    function setupKeyboardNavigationForEventCarouselSlideshow(divWrapperId, h2SlideshowPreviousId, h2SlideshowNextId, spanSlideshowPreviousId, spanSlideshowNextId) {
        
        if (!divWrapperId.startsWith('#')) { divWrapperId = '#' + divWrapperId; }
        if (!h2SlideshowPreviousId.startsWith('#')) { h2SlideshowPreviousId = '#' + h2SlideshowPreviousId; }
        if (!spanSlideshowPreviousId.startsWith('#')) { spanSlideshowPreviousId = '#' + spanSlideshowPreviousId; }
        if (!h2SlideshowNextId.startsWith('#')) { h2SlideshowNextId = '#' + h2SlideshowNextId; }
        if (!spanSlideshowNextId.startsWith('#')) { spanSlideshowNextId = '#' + spanSlideshowNextId; }

        let wrapperElem = $(divWrapperId)[0];
        if (!isEmpty($(wrapperElem).attr('data-cg-slider-init'))) {
            return; // already setup
        }
        $(wrapperElem).attr('data-cg-slider-init', 'true');

        var intervalId = setInterval(function () {

            let h2Prev = $(h2SlideshowPreviousId)[0];
            let h2Next = $(h2SlideshowNextId)[0];
            let spanPrev = $(spanSlideshowPreviousId)[0];
            let spanNext = $(spanSlideshowNextId)[0];

            if (isEmpty(h2Prev) || isEmpty(h2Next)) {
                clearInterval(intervalId);
            } else {
                if (isEmpty($(h2Prev).attr('data-cg-slickId'))) {
                    $(h2Prev).attr('data-cg-slickId', h2SlideshowPreviousId);
                    $(h2Next).attr('data-cg-slickId', h2SlideshowNextId);
                    $(spanPrev).attr('data-cg-slickId', spanSlideshowPreviousId);
                    $(spanNext).attr('data-cg-slickId', spanSlideshowNextId);

                    $(h2SlideshowPreviousId).keyup(function (event) {
                        if (event.which === 13 || event.which === 32) {
                            previousEventCarouselButtonOnClick(divWrapperId, h2SlideshowPreviousId);
                        }
                    });

                    $(h2SlideshowNextId).keyup(function (event) {
                        if (event.which === 13 || event.which === 32) {
                            $(h2SlideshowNextId).click();
                            setFocusOnEventsCarouselNextSlideButtonClick(divWrapperId, h2SlideshowNextId);
                        }
                    });

                    $(h2SlideshowPreviousId).on("focus", function () {
                        if (isAccKeyboardMode()) {
                            $(spanSlideshowPreviousId).removeClass('mdi-chevron-left');
                            $(spanSlideshowPreviousId).addClass('mdi-chevron-left-box');
                        }
                    });

                    $(h2SlideshowNextId).on("focus", function () {
                        if (isAccKeyboardMode()) {
                            $(spanSlideshowNextId).removeClass('mdi-chevron-right');
                            $(spanSlideshowNextId).addClass('mdi-chevron-right-box');
                        }
                    });

                    $(h2SlideshowPreviousId).on("blur", function () {
                        if (isAccKeyboardMode()) {
                            $(spanSlideshowPreviousId).removeClass('mdi-chevron-left-box');
                            $(spanSlideshowPreviousId).addClass('mdi-chevron-left');
                        }
                    });

                    $(h2SlideshowNextId).on("blur", function () {
                        if (isAccKeyboardMode()) {
                            $(spanSlideshowNextId).removeClass('mdi-chevron-right-box');
                            $(spanSlideshowNextId).addClass('mdi-chevron-right');
                        }
                    });

                    $(h2SlideshowPreviousId).attr('tabindex', '0');
                    $(h2SlideshowNextId).attr('tabindex', '0');
                }
            }
        }, 1000);
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle click/keyboard enter for Events carousel/slideshow PREVIOUS button
     * - Set focus to the first visible left card inside the slideshow after prev.click() udpates the visible slides
     * - When previous is clicked on the initial slideshow, or when previous is clicked on the first slideshow card, it takes a bit more time for slick to update the display
     * 
     * @param {string} divWrapperId           : ID of the slideshow container div
     * @param {string} h2SlideshowPreviousId  : ID of the slideshow previous button
     */
    function previousEventCarouselButtonOnClick(divWrapperId, h2SlideshowPreviousId) {
        if (!divWrapperId.startsWith('#')) { divWrapperId = '#' + divWrapperId; }
        if (!h2SlideshowPreviousId.startsWith('#')) { h2SlideshowPreviousId = '#' + h2SlideshowPreviousId; }
        $(divWrapperId + ' .event_slider').slick('slickPrev');
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle click/keyboard enter for Events carousel/slideshow NEXT button
     * - Set focus to the first visible right-most card inside the slideshow after prev.click() udpates the visible slides
     * 
     * @param {string} divWrapperId       : ID of the slideshow container div
     * @param {string} h2SlideshowNextId  : ID of the slideshow next button
     */
    function nextEventCarouselButtonOnClick(divWrapperId, h2SlideshowNextId) {
        if (!divWrapperId.startsWith('#')) { divWrapperId = '#' + divWrapperId; }
        if (!h2SlideshowNextId.startsWith('#')) { h2SlideshowNextId = '#' + h2SlideshowNextId; }
        $(divWrapperId + ' .event_slider').slick('slickNext');
    }

    /* --------------------------------------------------------------------------------------------------
     * Set focus to the left-most visible slide in the event carousel/slidewhow.
     * 
     * @param {string} divWrapperId           : ID of the slideshow container div
     * @param {string} h2SlideshowPreviousId  : ID of the slideshow previous button
     * 
     * NOTE h2SlideshowPreviousId does not appear to be used anymore (can remove in next refactoring)
     * 
     */
    function setFocusOnEventsCarouselPreviousSlideButtonClick(divWrapperId, h2SlideshowPreviousId) {
        if (!divWrapperId.startsWith('#')) { divWrapperId = '#' + divWrapperId; }
        if (!h2SlideshowPreviousId.startsWith('#')) { h2SlideshowPreviousId = '#' + h2SlideshowPreviousId; }

        setTimeout(function () {
            var slickActiveSlides = $(divWrapperId + ' div.slick-track div.slick-active');
            if (!isEmpty(slickActiveSlides) && slickActiveSlides.length > 0) {
                var focusCard = slickActiveSlides.get(0);
                if (isAccessibilityJSVerbose) {
                    console.log('> focusCard=' + getElementPath(focusCard));
                }
                $(focusCard).focus();
            }
        }, 100);
    }

    /* --------------------------------------------------------------------------------------------------
     * Set focus to the right-most visible slide in the event carousel/slidewhow.
     * 
     * @param {string} divWrapperId       : ID of the slideshow container div
     * @param {string} h2SlideshowNextId  : ID of the slideshow next button
     * 
     * NOTE h2SlideshowNextId does not appear to be used anymore (can remove in next refactoring)
     * 
     */
    function setFocusOnEventsCarouselNextSlideButtonClick(divWrapperId, h2SlideshowNextId) {
        if (!divWrapperId.startsWith('#')) { divWrapperId = '#' + divWrapperId; }
        if (!h2SlideshowNextId.startsWith('#')) { h2SlideshowNextId = '#' + h2SlideshowNextId; }

        setTimeout(function () {
            var slickActiveSlides = $(divWrapperId + ' div.slick-track div.slick-active');
            if (!isEmpty(slickActiveSlides) && slickActiveSlides.length > 0) {
                var focusCard = slickActiveSlides.get(slickActiveSlides.length - 1);
                if (isAccessibilityJSVerbose) {
                    console.log('> focusCard=' + getElementPath(focusCard));
                }
                $(focusCard).focus();
            }
        }, 100);
    }

/*-----------------------------------------------------*\
 * @CALENDAR / @DATEPICKER
 * 
 * Setup and manage keyboard navigation for datepicker controls.
 * 
\*-----------------------------------------------------*/

    var currentDateInput = null; // set to the currently active datepicker on focus
    var datePickerMadeAccessible = false; // will be set to true when keyboard setup is complete

    // daysOfWeek used to determine the day name of an associated datepicker cell based on what column it's in
    var daysOfWeek = [];
    daysOfWeek[0] = 'Sunday';
    daysOfWeek[1] = 'Monday';
    daysOfWeek[2] = 'Tuesday';
    daysOfWeek[3] = 'Wednesday';
    daysOfWeek[4] = 'Thursday';
    daysOfWeek[5] = 'Friday';
    daysOfWeek[6] = 'Saturday';

    /* --------------------------------------------------------------------------------------------------
     * Setup the datepicker keyboard support 
     * 
     */
    function accessibilityMagic() {
        setTimeout(function() {
            // Hide the "today" button because it doesn't do what
            // you think it supposed to do
            $(".ui-datepicker-current").hide();

            var container = document.getElementById('ui-datepicker-div');

            if (!container) {
                console.log("No container");
                return;
            }
            
            container.setAttribute('role', 'application');
            container.setAttribute('aria-label', 'Calendar view date-picker. Use arrow keys, page up, page down, home and end keys to change date. Press enter to select or escape to exit.');

            // the top controls:
            var prev = $('.ui-datepicker-prev', container)[0],
                next = $('.ui-datepicker-next', container)[0];


            // Lock the next/prev buttons from keyboard use - we don't need them (use arrow keys, page up/down, home, end instead); TAB/SHIFT-TAB will simply close the datepicker
            // NOTE the tab/shif-tab key will exit the datepicker (setting tabindex to 0 on the next/prev datepicker top corner controls does not fix this)
            next.href = 'javascript:;';
            next.setAttribute('tabindex', '-1');
            next.setAttribute('role', 'button');
            next.removeAttribute('title');

            prev.href = 'javascript:;';
            prev.setAttribute('tabindex', '-1');
            prev.setAttribute('role', 'button');
            prev.removeAttribute('title');

            appendOffscreenMonthText(next);
            appendOffscreenMonthText(prev);

            // delegation won't work here for whatever reason, so we are
            // forced to attach individual click listeners to the prev /
            // next month buttons each time they are added to the DOM
            $(next).on('click', handleNextClicks);
            $(prev).on('click', handlePrevClicks);

            monthDayYearText();
            
            datePickHandler();
            
            $(document).on('click', '#ui-datepicker-div .ui-datepicker-close', function () {
                closeCalendar();
            });
        });
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle keydown events while inside the datepicker (ESC, arrow up/down/left/right, etc.)
     * 
     */
    function datePickHandler() {
        var container = document.getElementById('ui-datepicker-div');
        if (!container) {
            console.log("No container");
            return;
        }
        var activeDate;
        var prev = $('.ui-datepicker-prev', container)[0],
            next = $('.ui-datepicker-next', container)[0];
            
        $('#ui-datepicker-div').on('keydown', function calendarKeyboardListener(keyVent) {
            var which = keyVent.which;
            var target = keyVent.target;
            var dateCurrent = getCurrentDate(container);

            if (!dateCurrent) {
                dateCurrent = $('a.ui-state-default')[0];
                setHighlightState(dateCurrent, container);
            }

            if (27 === which) {
                keyVent.stopPropagation();
                return closeCalendar();
            } else if (isShiftTabKey(keyVent)) {
                // Shift-TAB Key : we're not allowing keyboard access to the next/prev buttons and the close button is not displayed; just exit the datepicker
                keyVent.preventDefault();
                keyVent.stopPropagation();
                return closeCalendar();
            } else if (isTabKey(keyVent)) {
                // Forward TAB Key : we're not allowing keyboard access to the next/prev buttons and the close button is not displayed; just exit the datepicker
                keyVent.preventDefault();
                keyVent.stopPropagation();
                return closeCalendar();
            } else if (which === 37) { // LEFT arrow key
                // if we're on a date link...
                if (!$(target).hasClass('ui-datepicker-close') && $(target).hasClass('ui-state-default')) {
                    keyVent.preventDefault();
                    previousDay(target);
                }
            } else if (which === 39) { // RIGHT arrow key
                // if we're on a date link...
                if (!$(target).hasClass('ui-datepicker-close') && $(target).hasClass('ui-state-default')) {
                    keyVent.preventDefault();
                    nextDay(target);
                }
            } else if (which === 38) { // UP arrow key
                if (!$(target).hasClass('ui-datepicker-close') && $(target).hasClass('ui-state-default')) {
                    keyVent.preventDefault();
                    upHandler(target, container, prev);
                }
            } else if (which === 40) { // DOWN arrow key
                if (!$(target).hasClass('ui-datepicker-close') && $(target).hasClass('ui-state-default')) {
                    keyVent.preventDefault();
                    downHandler(target, container, next);
                }
            } else if (which === 13) { // ENTER
                if ($(target).hasClass('ui-state-default')) {
                    setTimeout(function () {
                        closeCalendar();
                    }, 100);
                } else if ($(target).hasClass('ui-datepicker-prev')) {
                    handlePrevClicks();
                } else if ($(target).hasClass('ui-datepicker-next')) {
                    handleNextClicks();
                }
            } else if (32 === which) {
                if ($(target).hasClass('ui-datepicker-prev') || $(target).hasClass('ui-datepicker-next')) {
                    target.click();
                }
            } else if (33 === which) { // PAGE UP
                keyVent.preventDefault();
                moveOneMonth(target, 'prev');
            } else if (34 === which) { // PAGE DOWN
                keyVent.preventDefault();
                moveOneMonth(target, 'next');
            } else if (36 === which) { // HOME
                keyVent.preventDefault();
                var firstOfMonth = $(target).closest('tbody').find('.ui-state-default')[0];
                if (firstOfMonth) {
                    firstOfMonth.focus();
                    setHighlightState(firstOfMonth, $('#ui-datepicker-div')[0]);
                }
            } else if (35 === which) { // END
                keyVent.preventDefault();
                var $daysOfMonth = $(target).closest('tbody').find('.ui-state-default');
                var lastDay = $daysOfMonth[$daysOfMonth.length - 1];
                if (lastDay) {
                    lastDay.focus();
                    setHighlightState(lastDay, $('#ui-datepicker-div')[0]);
                }
            }
            $(".ui-datepicker-current").hide();
        });
    }

    /* --------------------------------------------------------------------------------------------------
     * Close the datepicker control
     * 
     * NOTE: We can remove this function and just call $(currentDateInput).datepicker('hide'); if it's more clear to do so
     * 
     */
    function closeCalendar() {
        $(currentDateInput).datepicker('hide');
    }

    /* --------------------------------------------------------------------------------------------------
     * Move the calender forward or backwards one month (next/prev month) when the user presses PAGE-UP or PAGE-DOWN keys
     * 
     * @param {HTMLElement} currentDate  : The link <a> representing the currently highlighted date (received keyboard event)
     * @param {string}      dir          : The direction to move 'prev' / 'next'
     * 
     */
    function moveOneMonth(currentDate, dir) {
        var button = (dir === 'next')
            ? $('.ui-datepicker-next')[0]
            : $('.ui-datepicker-prev')[0];

        if (!button) {
            return;
        }

        var ENABLED_SELECTOR = '#ui-datepicker-div tbody td:not(.ui-state-disabled)';
        var $currentCells = $(ENABLED_SELECTOR);
        var currentIdx = $.inArray(currentDate.parentNode, $currentCells);

        button.click();
        setTimeout(function () {
            updateHeaderElements();

            var $newCells = $(ENABLED_SELECTOR);
            var newTd = $newCells[currentIdx];
            var newAnchor = newTd && $(newTd).find('a')[0];

            while (!newAnchor) {
                currentIdx--;
                newTd = $newCells[currentIdx];
                newAnchor = newTd && $(newTd).find('a')[0];
            }

            setHighlightState(newAnchor, $('#ui-datepicker-div')[0]);
            newAnchor.focus();

        }, 0);

    }

    /* --------------------------------------------------------------------------------------------------
     * Handle clicks from the datepicker's [previous] button (top-left of the calendare title bar)
     * 
     */
    function handlePrevClicks() {
        setTimeout(function () {
            updateHeaderElements();
            prepHighlightState();
            $('.ui-datepicker-prev').trigger("focus");
            $(".ui-datepicker-current").hide();
        }, 0);
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle clicks from the datepicker's [next] button (top-right of the calendare title bar)
     * 
     */
    function handleNextClicks() {
        setTimeout(function () {
            updateHeaderElements();
            prepHighlightState();
            $('.ui-datepicker-next').trigger("focus");
            $(".ui-datepicker-current").hide();
        }, 0);
    }

    /* --------------------------------------------------------------------------------------------------
     * Handles left arrow key navigation.
     * Attempt to move to the previous day. If not available, move to the previous week.
     * 
     * @param {HTMLElement} dateLink  : The link <a> representing the currently highlighted date (received keyboard event)
     * 
     */
    function previousDay(dateLink) {
        var container = document.getElementById('ui-datepicker-div');
        if (!dateLink) {
            return;
        }
        var td = $(dateLink).closest('td');
        if (!td) {
            return;
        }

        var prevTd = $(td).prev(),
            prevDateLink = $('a.ui-state-default', prevTd)[0];

        if (prevTd && prevDateLink) {
            setHighlightState(prevDateLink, container);
            prevDateLink.focus();
        } else {
            handlePrevious(dateLink);
        }
    }


    /* --------------------------------------------------------------------------------------------------
     * Attempt to move to the previous week. If not possible, move to the previous month.
     * 
     * @param {HTMLElement} target  : The link <a> representing the currently highlighted date (received keyboard event)
     * 
     */
    function handlePrevious(target) {
        var container = document.getElementById('ui-datepicker-div');
        if (!target) {
            return;
        }
        var currentRow = $(target).closest('tr');
        if (!currentRow) {
            return;
        }
        var previousRow = $(currentRow).prev();

        if (!previousRow || previousRow.length === 0) {
            // there is not previous row, so we go to previous month...
            previousMonth();
        } else {
            var prevRowDates = $('td a.ui-state-default', previousRow);
            var prevRowDate = prevRowDates[prevRowDates.length - 1];

            if (prevRowDate) {
                setTimeout(function () {
                    setHighlightState(prevRowDate, container);
                    prevRowDate.focus();
                }, 0);
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Move the calendar to the previous month.
     * 
     */
    function previousMonth() {
        var prevLink = $('.ui-datepicker-prev')[0];
        var container = document.getElementById('ui-datepicker-div');
        prevLink.click();
        // focus last day of new month
        setTimeout(function () {
            var trs = $('tr', container),
                lastRowTdLinks = $('td a.ui-state-default', trs[trs.length - 1]),
                lastDate = lastRowTdLinks[lastRowTdLinks.length - 1];

            // updating the cached header elements
            updateHeaderElements();

            setHighlightState(lastDate, container);
            lastDate.focus();

        }, 0);
    }

    /* --------------------------------------------------------------------------------------------------
     * Handles right arrow key navigation.
     * Attempt to move to the next day. If not available, move to the next week.
     * 
     * @param {HTMLElement} dateLink  : The link <a> representing the currently highlighted date (received keyboard event)
     * 
     */
    function nextDay(dateLink) {
        var container = document.getElementById('ui-datepicker-div');
        if (!dateLink) {
            return;
        }
        var td = $(dateLink).closest('td');
        if (!td) {
            return;
        }
        var nextTd = $(td).next(),
            nextDateLink = $('a.ui-state-default', nextTd)[0];

        if (nextTd && nextDateLink) {
            setHighlightState(nextDateLink, container);
            nextDateLink.focus(); // the next day (same row)
        } else {
            handleNext(dateLink);
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Attempt to move to the next week. If not possible, move to the next month.
     * 
     * @param {HTMLElement} target  : The link <a> representing the currently highlighted date (received keyboard event)
     * 
     */
    function handleNext(target) {
        var container = document.getElementById('ui-datepicker-div');
        if (!target) {
            return;
        }
        var currentRow = $(target).closest('tr'),
            nextRow = $(currentRow).next();

        if (!nextRow || nextRow.length === 0) {
            nextMonth();
        } else {
            var nextRowFirstDate = $('a.ui-state-default', nextRow)[0];
            if (nextRowFirstDate) {
                setHighlightState(nextRowFirstDate, container);
                nextRowFirstDate.focus();
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Move the calendar to the next month.
     * 
     */
    function nextMonth() {
        nextMon = $('.ui-datepicker-next')[0];
        var container = document.getElementById('ui-datepicker-div');
        nextMon.click();
        // focus the first day of the new month
        setTimeout(function () {
            // updating the cached header elements
            updateHeaderElements();

            var firstDate = $('a.ui-state-default', container)[0];
            setHighlightState(firstDate, container);
            firstDate.focus();
        }, 0);
    }

    /* --------------------------------------------------------------------------------------------------
     * Handles up arrow navigation through dates in calendar.
     * Attempt to move up one week on the same day. If not possible, move to the previous month (same day)
     * 
     * @param  {HTMLElement} target    : The link <a> representing the currently highlighted date (received keyboard event)
     * @param  {Element}     cont      : The calendar container
     * @param  {HTMLElement} prevLink  : Link to navigate to previous month
     * 
     */
    function upHandler(target, cont, prevLink) {
        prevLink = $('.ui-datepicker-prev')[0];
        var rowContext = $(target).closest('tr');
        if (!rowContext) {
            return;
        }
        var rowTds = $('td', rowContext),
            rowLinks = $('a.ui-state-default', rowContext),
            targetIndex = $.inArray(target, rowLinks),
            prevRow = $(rowContext).prev(),
            prevRowTds = $('td', prevRow),
            parallel = prevRowTds[targetIndex],
            linkCheck = $('a.ui-state-default', parallel)[0];

        if (prevRow && parallel && linkCheck) {
            // there is a previous row, a td at the same index
            // of the target AND theres a link in that td
            setHighlightState(linkCheck, cont);
            linkCheck.focus();
        } else {
            // we're either on the first row of a month, or we're on the
            // second and there is not a date link directly above the target
            prevLink.click();
            setTimeout(function () {
                // updating the cached header elements
                updateHeaderElements();
                var newRows = $('tr', cont),
                    lastRow = newRows[newRows.length - 1],
                    lastRowTds = $('td', lastRow),
                    tdParallelIndex = $.inArray(target.parentNode, rowTds),
                    newParallel = lastRowTds[tdParallelIndex],
                    newCheck = $('a.ui-state-default', newParallel)[0];

                if (lastRow && newParallel && newCheck) {
                    setHighlightState(newCheck, cont);
                    newCheck.focus();
                } else {
                    // theres no date link on the last week (row) of the new month
                    // meaning its an empty cell, so we'll try the 2nd to last week
                    var secondLastRow = newRows[newRows.length - 2],
                        secondTds = $('td', secondLastRow),
                        targetTd = secondTds[tdParallelIndex],
                        linkCheck = $('a.ui-state-default', targetTd)[0];

                    if (linkCheck) {
                        setHighlightState(linkCheck, cont);
                        linkCheck.focus();
                    }

                }
            }, 0);
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Handles down arrow navigation through dates in calendar.
     * Attempt to move down one week on the same day. If not possible, move to the next month (same day)
     * 
     * @param  {HTMLElement} target    : The link <a> representing the currently highlighted date (received keyboard event)
     * @param  {Element}     cont      : The calendar container
     * @param  {HTMLElement} nextLink  : Link to navigate to next month
     * 
     */
    function downHandler(target, cont, nextLink) {
        nextLink = $('.ui-datepicker-next')[0];
        var targetRow = $(target).closest('tr');
        if (!targetRow) {
            return;
        }
        var targetCells = $('td', targetRow),
            cellIndex = $.inArray(target.parentNode, targetCells), // the td (parent of target) index
            nextRow = $(targetRow).next(),
            nextRowCells = $('td', nextRow),
            nextWeekTd = nextRowCells[cellIndex],
            nextWeekCheck = $('a.ui-state-default', nextWeekTd)[0];

        if (nextRow && nextWeekTd && nextWeekCheck) {
            // theres a next row, a TD at the same index of `target`,
            // and theres an anchor within that td
            setHighlightState(nextWeekCheck, cont);
            nextWeekCheck.focus();
        } else {
            nextLink.click();

            setTimeout(function () {
                // updating the cached header elements
                updateHeaderElements();

                var nextMonthTrs = $('tbody tr', cont),
                    firstTds = $('td', nextMonthTrs[0]),
                    firstParallel = firstTds[cellIndex],
                    firstCheck = $('a.ui-state-default', firstParallel)[0];

                if (firstParallel && firstCheck) {
                    setHighlightState(firstCheck, cont);
                    firstCheck.focus();
                } else {
                    // lets try the second row b/c we didnt find a
                    // date link in the first row at the target's index
                    var secondRow = nextMonthTrs[1],
                        secondTds = $('td', secondRow),
                        secondRowTd = secondTds[cellIndex],
                        secondCheck = $('a.ui-state-default', secondRowTd)[0];

                    if (secondRow && secondCheck) {
                        setHighlightState(secondCheck, cont);
                        secondCheck.focus();
                    }
                }
            }, 0);
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Determine the correct highligh position, then set the highlight
     * 
     */
    function prepHighlightState() {
        var highlight;
        var cage = document.getElementById('ui-datepicker-div');
        highlight = $('.ui-state-highlight', cage)[0] ||
            $('.ui-state-default', cage)[0];
        if (highlight && cage) {
            setHighlightState(highlight, cage);
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Set the highlighted class to date elements, when focus is received
     * 
     * @param  {HTMLElement} newHighlight  : The link <a> representing the new focus target (to receive keyboard focus; date to be highlighted)
     * @param  {Element}     container     : The conainer DIV element for this datepicker control
     */
    function setHighlightState(newHighlight, container) {
        var prevHighlight = getCurrentDate(container);
        // remove the highlight state from previously
        // highlighted date and add it to our newly active date
        $(prevHighlight).removeClass('ui-state-highlight');
        $(newHighlight).addClass('ui-state-highlight');
    }

    /* --------------------------------------------------------------------------------------------------
     * add an aria-label to the date link indicating the currently focused date
     * (formatted identically to the required format: mm/dd/yyyy)
     * 
     * NOTE this may be overridden by code below; review and make sure we're only doing this once and doing it correctly
     * 
     */
    function monthDayYearText() {
        var cleanUps = $('.amaze-date');

        $(cleanUps).each(function (clean) {
            // each(cleanUps, function (clean) {
            clean.parentNode.removeChild(clean);
        });

        var datePickDiv = document.getElementById('ui-datepicker-div');
        // in case we find no datepick div
        if (!datePickDiv) {
            return;
        }

        var dates = $('a.ui-state-default', datePickDiv);
        $(dates).attr('role', 'button').on('keydown', function (e) {
            if (e.which === 32) {
                e.preventDefault();
                e.target.click();
                setTimeout(function () {
                    closeCalendar();
                }, 100);
            }
        });
        $(dates).each(function (index, date) {
            var currentRow = $(date).closest('tr'),
                currentTds = $('td', currentRow),
                currentIndex = $.inArray(date.parentNode, currentTds),
                headThs = $('thead tr th', datePickDiv),
                dayIndex = headThs[currentIndex],
                daySpan = $('span', dayIndex)[0],
                monthName = $('.ui-datepicker-month', datePickDiv)[0].innerHTML,
                year = $('.ui-datepicker-year', datePickDiv)[0].innerHTML,
                number = date.innerHTML;

            if (!daySpan || !monthName || !number || !year) {
                return;
            }

            // AT Reads: {month} {date} {year} {day}
            // "December 18 2014 Thursday"
            var dateText = date.innerHTML + ' ' + monthName + ' ' + year + ' ' + daySpan.title;
            // AT Reads: {date(number)} {name of day} {name of month} {year(number)}
            // var dateText = date.innerHTML + ' ' + daySpan.title + ' ' + monthName + ' ' + year;
            // add an aria-label to the date link reading out the currently focused date
            date.setAttribute('aria-label', dateText);
        });
    }

    /* --------------------------------------------------------------------------------------------------
     * update the cached header elements because we're in a new month or year
     * 
     */
    function updateHeaderElements() {
        var context = document.getElementById('ui-datepicker-div');
        if (!context) {
            return;
        }

        //  $(context).find('table').first().attr('role', 'grid');

        prev = $('.ui-datepicker-prev', context)[0];
        next = $('.ui-datepicker-next', context)[0];

        //make them click/focus - able
        //next.href = 'javascript:;';
        //prev.href = 'javascript:;';

        //next.setAttribute('role', 'button');
        //prev.setAttribute('role', 'button');
        appendOffscreenMonthText(next);
        appendOffscreenMonthText(prev);

        //$(next).on('click', handleNextClicks);
        //$(prev).on('click', handlePrevClicks);

        // add month day year text
        monthDayYearText();
    }


    /* --------------------------------------------------------------------------------------------------
     * Appends logical next/prev month text to the buttons
     * - ex: Next Month, January 2015
     *       Previous Month, November 2014
     * 
     * @param  {Element} button  : The button to which the off screen month text is to be added
     * 
     */
    function appendOffscreenMonthText(button) {
        var buttonText;
        var isNext = $(button).hasClass('ui-datepicker-next');
        var months = [
            'january', 'february',
            'march', 'april',
            'may', 'june', 'july',
            'august', 'september',
            'october',
            'november', 'december'
        ];

        var currentMonth = $('.ui-datepicker-title .ui-datepicker-month').text().toLowerCase();
        var monthIndex = $.inArray(currentMonth.toLowerCase(), months);
        var currentYear = $('.ui-datepicker-title .ui-datepicker-year').text().toLowerCase();
        var adjacentIndex = (isNext) ? monthIndex + 1 : monthIndex - 1;

        if (isNext && currentMonth === 'december') {
            currentYear = parseInt(currentYear, 10) + 1;
            adjacentIndex = 0;
        } else if (!isNext && currentMonth === 'january') {
            currentYear = parseInt(currentYear, 10) - 1;
            adjacentIndex = months.length - 1;
        }

        buttonText = (isNext)
            ? 'Next Month, ' + firstToCap(months[adjacentIndex]) + ' ' + currentYear
            : 'Previous Month, ' + firstToCap(months[adjacentIndex]) + ' ' + currentYear;

        $(button).find('.ui-icon').html(buttonText);

    }

    /* --------------------------------------------------------------------------------------------------
     * grabs the current date based on the highlight class
     * 
     * @param  {Element} container  : The conainer DIV element for this datepicker control
     * 
     */
    function getCurrentDate(container) {
        var currentDate = $('.ui-state-highlight', container)[0];
        return currentDate;
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle jumping forward over the next datepicker by pressing the CTRL key
     * When datepicker is surrounded by "skip" controls, user can press the CTRL key to jump over the datepicker (instead of entering/exiting the datepicker)
     * 
     * For datepickers in the filter bar, the current implementation is <span><div/datepicker><span> where each span jumps to the other when pressing CTRL
     * For datepickers in forms, or in vertical UI (navigating top to bottom) the current implementation is <div><div/datepicker><div> where the surrounding div's skip to each other on CTRL
     * 
     * @param  {KeyboardEvent} event    : Keydown event on the target
     * @param  {string}        focusId  : Focus target if the event represents a CTRL keydown (typically the sibling skip-link on the other side of the datepicker control)
     * 
     */
    function aboveDatePickerOnKeyDown(event, focusId) {
        if (event.which == 17) {
            // CTRL key pressed
            var focusElem = document.getElementById(focusId);
            if (isEmpty(focusElem)) {
                console.error('focusElem empty for focusId=' + focusId);
            } else {
                focusElem.focus();
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Handle jumping backwards over the previous datepicker by pressing the CTRL key
     * When datepicker is surrounded by "skip" controls, user can press the CTRL key to jump over the datepicker (instead of entering/exiting the datepicker)
     * 
     * For datepickers in the filter bar, the current implementation is <span><div/datepicker><span> where each span jumps to the other when pressing CTRL
     * For datepickers in forms, or in vertical UI (navigating top to bottom) the current implementation is <div><div/datepicker><div> where the surrounding div's skip to each other on CTRL
     * 
     * @param  {KeyboardEvent} event    : Keydown event on the target
     * @param  {string}        focusId  : Focus target if the event represents a CTRL keydown (typically the sibling skip-link on the other side of the datepicker control)
     * 
     */
    function belowDatePickerOnKeyDown(event, focusId) {
        if (event.which == 17) {
            // CTRL key pressed
            var focusElem = document.getElementById(focusId);
            if (isEmpty(focusElem)) {
                console.error('focusElem empty for focusId=' + focusId);
            } else {
                focusElem.focus();
            }
        }
    }

    /* --------------------------------------------------------------------------------------------------
     * Show the datepicker (expand/make visible) when it receives focus
     * 
     * NOTE if we permanently require no more work than calling $(datePickerId).datepicker('show'); we can remove this function and just make the call directly
     * 
     * @param  {string}  datePickerId  : ID of the datepicker (container div)
     * 
     */
    function datePickerReShow(datePickerId) {
        $(datePickerId).datepicker('show');
    }

    /* --------------------------------------------------------------------------------------------------
     * Restart the datepicker when it closed. Setup the focus handler and wait to receive focus again.
     * 
     * @param  {string}  datePickerId  : ID of the datepicker (container div)
     * 
     */
    function datePickerRestart(datePickerId) {
        // need to wait a few millis before restart (let other DOM tasks finish up)
        setTimeout(function () {
            $(datePickerId).on('focus', function() {
                //
                // need to disable the onFocus of the associated input first or it will be called continously (will be reset on datepicker exit/close)
                //
                $(datePickerId).off('focus');
                datePickerReShow(datePickerId);
            });
        }, 500);
    }

    /* --------------------------------------------------------------------------------------------------
     * Close the datepicker and setup to receive future focus.
     * 
     * @param  {string}  datePickerId  : ID of the datepicker (container div)
     * @param  {string}  dateText      : The date selected.
     * @param  {string}  focusId       : ID of the target element to receive focus after closing the datepicker
     * 
     */
    function datePickerOnClose(datePickerId, dateText, focusId) {
        if (isEmpty(datePickerId)) { console.error('datePickerId is empty'); return; }
        if (isEmpty(focusId)) { console.error('focusId is empty'); return; }
        if (!datePickerId.startsWith('#')) { datePickerId = '#' + datePickerId; }
        if (!focusId.startsWith('#')) { focusId = '#' + focusId; }
        if (!isEmpty(dateText)) {
            $(datePickerId).attr('value', dateText);
            $(datePickerId).datepicker('setDate', dateText);
        }
        if ($(focusId).length > 0) {
            $(focusId)[0].focus();
        }
        $(datePickerId).removeClass('hasMonitor');
        datePickerRestart(datePickerId);
    }

    /* --------------------------------------------------------------------------------------------------
     * Monitor the datepicker while open and update required elements/attributes when we change months
     * 
     * @param  {string}  datePickerId  : ID of the datepicker (container div)
     * @param  {string}  initialDate   : The currently selected date when starting/opening the datepicker
     */
    function monitorDatePicker(datePickerId, initialDate) {
        if (isEmpty(datePickerId)) { console.error('datePickerId is empty.'); return; }
        if (!datePickerId.startsWith('#')) { datePickerId = '#' + datePickerId; }
        if (isEmpty(initialDate)) { initialDate = null; }

        if ($(datePickerId).hasClass('hasMonitor')) { return; }
        $(datePickerId).addClass('hasMonitor');

        var waitSeconds = 5;
        var epochStart = Date.now();
        var intervalId = setInterval(function () {
            if (!$(datePickerId).hasClass('hasMonitor')) {
                clearInterval(intervalId);
                return;
            }
            var datePickerDivs = $('#ui-datepicker-div');
            if (!isEmpty(datePickerDivs) && datePickerDivs.length > 0) {
                for (var i=0; i<datePickerDivs.length; i++) {
                    if (datePickerDivs[i].style.display == 'none') {
                        $(datePickerId).removeClass('hasMonitor');
                        clearInterval(intervalId);
                        return;
                    } else {
                        var isWorkRequired = true;
                        var datePickerTable = $('#ui-datepicker-div table');
                        if (datePickerTable !== undefined && datePickerTable.length > 0) {
                            if (datePickerTable[0].getAttribute('role') === 'presentation') {
                                isWorkRequired = false;
                            }
                        }
                        if (isWorkRequired) {
                            var focusElem = null;
                            datePickerTable[0].setAttribute('role', 'presentation');

                            var datePickerTitleMonths = $('div.ui-datepicker-title span.ui-datepicker-month');
                            if (datePickerTitleMonths !== undefined && datePickerTitleMonths.length > 0) {
                                var monthName = datePickerTitleMonths[0].textContent;
                            }
                            var datePickerTitleYears = $('div.ui-datepicker-title span.ui-datepicker-year');
                            if (datePickerTitleYears !== undefined && datePickerTitleYears.length > 0) {
                                var yearNumber = datePickerTitleYears[0].textContent;
                            }
                            var dataPickerRows = $(datePickerTable).find('> tbody > tr');
                            if (dataPickerRows !== undefined && dataPickerRows.length > 0) {
                                for (var row=0; row<dataPickerRows.length; row++) {
                                    var tds = $(dataPickerRows[row]).find('td');
                                    if (tds !== undefined && tds.length > 0) {
                                        for (var cell=0; cell<tds.length; cell++) {
                                            var links = $(tds[cell]).find('a');
                                            if (links !== undefined && links.length > 0) {
                                                var dayOfWeek = daysOfWeek[cell];
                                                var dayOfMonth = links[0].textContent;
                                                var dayOfMonthText = '' + dayOfMonth;
                                                if (dayOfMonthText.length < 2) {
                                                    dayOfMonthText = '0' + dayOfMonthText;
                                                }
                                                var ariaLabel = '' + dayOfMonth + ' ' + monthName + ' ' + yearNumber + ' ' + dayOfWeek;
                                                var datePickerDateText1 = dayOfMonthText + ' ' + monthName.substring(0, 3) + ' ' + yearNumber;
                                                var datePickerDateText2 = dayOfMonthText + ' ' + monthName.substring(0, 3) + ' ' + (yearNumber % 100);
                                                if (!isEmpty(initialDate)) {
                                                    if (initialDate === datePickerDateText1) {
                                                        focusElem = links[0];
                                                        initialDate = null;
                                                    } else if (initialDate === datePickerDateText2) {
                                                        focusElem = links[0];
                                                        initialDate = null;
                                                    }
                                                }
                                                links[0].setAttribute('aria-label', ariaLabel);
                                            }
                                        }
                                    }
                                }
                            }
                            if (!isEmpty(focusElem)) {
                                focusElem.focus();
                            }
                        }
                    }
                }
            } else {
                // datepicker element not found
                var deltaSeconds = getElapsedSeconds(epochStart);
                if (deltaSeconds > waitSeconds) {
                    $(datePickerId).removeClass('hasMonitor');
                    clearInterval(intervalId);
                }
            }
        }, 100);
    }

    /* --------------------------------------------------------------------------------------------------
     * Setup/initialize the datepicker for use in our environment
     * - set onClose handler
     * - set focusin handler
     * 
     */
    (function($) {
        if ($.datepicker) {
            $.datepicker.setDefaults({
                showOn: "",
                onClose: function(date, input) {
                    console.log(input);
                    $(input).focus();
                    currentDateInput = undefined;
                }
            });
        }

        var oldDatepicker = $.fn.datepicker;
         $.fn.datepicker = function()
        {
            var ret = oldDatepicker.apply(this, arguments);

            this.focusin(function(ev) {
                if (currentDateInput != ev.target) {
                    if (!datePickerMadeAccessible) {
                        accessibilityMagic();
                        datePickerMadeAccessible = true;
                    }
                    
                    $(ev.target).datepicker('show');
                    
                    currentDateInput = ev.target;
                    var today = $('.ui-datepicker-today a')[0];

                    if (!today) {
                        today = $('.ui-state-active')[0] ||
                            $('.ui-state-default')[0];
                    }
                    if (today) { today.focus(); }
                }
            });

            return ret;
        };
    })(jQuery);

/*-----------------------------------------------------*\
 * @FEEDS | @FEEDPOSTS
 * 
 * Functions to manage accessibility for feed post tabs
 * and feed post content.
 * 
\*-----------------------------------------------------*/

/* Function that flags the currently selected tab as aria-selected=true (and all other tabs to false)
 * @param {string} containerId    : the ID of the element containing the tabs
 * @param {string} selectedLinkId : the ID of the tab link just clicked
 */
function flagSelectedTab(containerId, selectedLinkId) {
    $('#' + containerId + ' a').attr('aria-selected','false');
    $('#' + selectedLinkId).attr('aria-selected','true');
}

/* Function that adds additional real-time information to accessibility labels (aria-label)
 * ex. feed posts (both on load after aspx has processed and on "load more" scroll beyond bottom)
 */
function processAriaLive() {
    let intervalId = setInterval(function () {
        let ariaLiveElements = $('[data-aria-live]');
        for (var i=0; i<ariaLiveElements.length; i++) {
            let ariaLiveElement = ariaLiveElements[i];
            let ariaContainers = $(ariaLiveElement).closest('[data-aria-container]');
            if (ariaContainers.length > 0) {
                let ariaContainerLabel = $(ariaContainers[0]).attr('aria-label');
                if (isEmpty(ariaContainerLabel)) {
                    ariaContainerLabel = '';
                }
                let linkAccLabel = $(ariaLiveElement).attr('aria-label') + ' ' + ariaContainerLabel;
                $(ariaLiveElement).attr('aria-label', getSafeAriaLabelTextTrim(linkAccLabel));
                $(ariaLiveElement).removeAttr('data-aria-live');
            }
        }
    }, 1000);
}

/* Function that assigns accessibility labels to the list checkboxes (left on each list item)
 * so the screen reader will read a unique label for each checkbox. Note that pages require setup
 * (currently only configured for Group Dashboard > Officers)
 */
function setupStrCheckboxForLists() {
    setTimeout(function() {
        let strCheckboxContainers = $('div.checkbox[data-acc-container]');
        for (var i=0; i<strCheckboxContainers.length; i++) {
            let checkboxContainer = strCheckboxContainers[i];
            let checkboxContainerAccLabel = $(checkboxContainer).attr('data-acc-label');
            if (!isEmpty(checkboxContainerAccLabel)) {
                let checkboxInputs = $(checkboxContainer).children('input');
                if (checkboxInputs.length > 0) {
                    $(checkboxInputs).attr('aria-label', checkboxContainerAccLabel);
                }
            }
        }
    }, 999); // wait for the page to fully load - users will not tab to here in under a second
}

/* Function that fixes the background color on the Tracks and Checklists pages (white text on background with some color) */
function updateTracksAndChecklistsBackgroundColors() {
    let doWork = false;
    let isWorking = false;
    let intervalCount = 0;
    let intervalId = setInterval(function () {
        intervalCount++;
        if (intervalCount > 150) {
            clearInterval(intervalId);
        }
        let trackHeaders = document.querySelectorAll('div.track__header');
        if (doWork && !isWorking) {
            isWorking = true;
            for (var i=0; i<trackHeaders.length; i++) {
                let trackHeader = trackHeaders[i];
                if (isEmpty(trackHeader.getAttribute('data-ccr'))) {
                    let backgroundColor = trackHeader.style.backgroundColor;
                    if (isEmpty(backgroundColor)) {
                        backgroundColor = '#087eb4'; // default for tracks and checklists
                        trackHeader.style.background = backgroundColor;
                    }
                    backgroundColor = getVisibleRGB(backgroundColor, '#ffffff');
                    if (getColorContrastRatio('#ffffff', backgroundColor) < 4.5) {
                        trackHeader.classList.add('dark-text');
                    }
                    trackHeader.setAttribute('data-ccr', 'ok');
                }
            }
            clearInterval(intervalId);
        }
        if (trackHeaders.length > 0) {
            doWork = true;
        }
    }, 83);
}

/* Function that fixes the background color on the user/member tags (white text on background with some color) */
function updateUserTagsBackgroundColors() {
    let doWork = false;
    let isWorking = false;
    let intervalCount = 0;
    let intervalId = setInterval(function () {
        intervalCount++;
        if (intervalCount > 150) {
            clearInterval(intervalId);
        }
        let tags = document.querySelectorAll('.label.label-tag');
        if (doWork && !isWorking) {
            isWorking = true;
            for (var i=0; i<tags.length; i++) {
                let tag = tags[i];
                if (isEmpty(tag.getAttribute('data-ccr'))) {
                    let computedBackgroundColor = window.getComputedStyle(tag).backgroundColor;
                    if (isEmpty(computedBackgroundColor)) {
                        computedBackgroundColor = tag.style.backgroundColor;
                        if (isEmpty(computedBackgroundColor)) {
                            computedBackgroundColor = '#767676'; // default grey
                            tag.style.backgroundColor = computedBackgroundColor;
                        }
                    }
                    computedBackgroundColor = getVisibleRGB(computedBackgroundColor, '#ffffff');
                    if (getColorContrastRatio('#ffffff', computedBackgroundColor) < 4.5) {
                        tag.classList.add('dark-text');
                    }                        
                    tag.setAttribute('data-ccr', 'ok');
                }
            }
            clearInterval(intervalId);
        }
        if (tags.length > 0) {
            doWork = true;
        }
    }, 79);
}

/* Function that fixes the background color on badges (white text on background with some color) */
function updateBadgeBackgroundColors() {
    let doWork = false;
    let isWorking = false;
    let intervalCount = 0;
    let intervalId = setInterval(function () {
        intervalCount++;
        if (intervalCount > 150) {
            clearInterval(intervalId);
        }
        let badges = document.querySelectorAll('a.badge');
        if (doWork && !isWorking) {
            isWorking = true;
            for (var i=0; i<badges.length; i++) {
                let badge = badges[i];
                if (isEmpty(badge.getAttribute('data-ccr'))) {
                    let computedBackgroundColor = window.getComputedStyle(badge).backgroundColor;
                    if (isEmpty(computedBackgroundColor)) {
                        computedBackgroundColor = badge.style.backgroundColor;
                        if (isEmpty(computedBackgroundColor)) {
                            computedBackgroundColor = '#767676'; // default grey
                            badge.style.backgroundColor = computedBackgroundColor;
                        }
                    }
                    computedBackgroundColor = getVisibleRGB(computedBackgroundColor, '#ffffff');
                    if (getColorContrastRatio('#ffffff', computedBackgroundColor) < 4.5) {
                        badge.classList.add('dark-text');
                    }                        
                    badge.setAttribute('data-ccr', 'ok');
                }
            }
            clearInterval(intervalId);
        }
        if (badges.length > 0) {
            doWork = true;
        }
    }, 83);
}

/* Function that closes a dropdown automatically when keyboard focus has left the dropdown group.
 * The dropdown group includes the dropdown button or "More" button that often has an icon with 3 vertical dots (class="caret") and not text,
 * as well as the associated dropdown list. When navigating with the keyboard, we want to automatically close this dropdown
 * when the user has left the dropdown list or the associated dropdown toggle button.
 */
function dropdownMenuFocusoutA11yHandler(elem) {
    if (isAccKeyboardMode()) {
        setTimeout(function () {
                let focusElem = document.activeElement;
                let isInsideBtnGroup = ($(focusElem).parents('.btn-group.open').length > 0) ? true : false;
                let isInsideDropdown = ($(focusElem).parents('.dropdown-menu').length > 0) ? true : false;
                if (!isInsideBtnGroup) {
                    $(elem).removeClass('open');
                    $(elem).find('[aria-expanded]').attr('aria-expanded', 'false');
                } else if (isInsideBtnGroup && !isInsideDropdown) {
                    $(elem).removeClass('open');
                    $(elem).find('[aria-expanded]').attr('aria-expanded', 'false');
                }
        }, 667);
    }
}

/*-----------------------------------------------------*\
    @ARIA-LIVE
    Status messages and updates specifically for the screen reader
\*-----------------------------------------------------*/

function a11yAnnounceToScreenReader(message) {
    // Create or get the aria-live region
    let liveRegion = document.getElementById('cg-aria-live-region');
    if (liveRegion) {
        liveRegion.textContent = message;
    } else {
        // If the live region doesn't already exist we will need time for the screen reader to recognize it and register it's current (empty) contents
        liveRegion = document.createElement('div');
        liveRegion.id = 'cg-aria-live-region';
        liveRegion.setAttribute('aria-live', 'polite');
        liveRegion.setAttribute('aria-atomic', 'true');
        liveRegion.style.position = 'absolute';
        liveRegion.style.left = '-9999px';
        document.body.appendChild(liveRegion);
        setTimeout(function() {
            a11yAnnounceToScreenReader(message);
        }, 333); // wait 1/3 second for the screen reader to register new live region
    }
}

/* Function that forces focus to the specified element (by cssSelector) with a screen reader announcement when complete.
 * For example: adding tags using the multiselect dropdown will submit the DB request then remove and re-define the DOM for the associated select,
 * and we want to force keyboard focus back to the select when it appears back in the DOM after DB update */
function a11yFocusAfterReset(cssSelector, announcement, timeoutSeconds) {
    $(cssSelector).attr('data-far', 'true');
    let waitForSelector = cssSelector + ':not([data-far="true"])';
    function focusAfterReset(focusElem) {
        if (focusElem) {
            focusElem.focus();
            a11yAnnounceToScreenReader(announcement);
        }
    }
    waitFor(waitForSelector, focusAfterReset, 111, timeoutSeconds * 1000);
}

/*-----------------------------------------------------*\
    @TRACKS AND CHECKLISTS
    Keyboard navigation and accessibility support for tracks and checklists
\*-----------------------------------------------------*/

function setupTracksAndChecklistsAccessibility() {
    // Add keyboard support for track toggle buttons
    $('.btn-icon[onclick*="toggleTrackChecklists"]').each(function() {
        $(this).attr('tabindex', '0');
        $(this).keydown(function(e) {
            if (e.keyCode === 13 || e.keyCode === 32) { // Enter or Space
                e.preventDefault();
                $(this).click();
            }
        });
    });

    // Add keyboard support for checklist toggle buttons
    $('.btn-icon[onclick*="toggleChecklist"]').each(function() {
        $(this).attr('tabindex', '0');
        $(this).keydown(function(e) {
            if (e.keyCode === 13 || e.keyCode === 32) { // Enter or Space
                e.preventDefault();
                $(this).click();
            }
        });
    });

    // Add keyboard support for checklist items
    $('.checklist-item input[type="checkbox"]').each(function() {
        $(this).attr('tabindex', '0');
        $(this).keydown(function(e) {
            if (e.keyCode === 13 || e.keyCode === 32) { // Enter or Space
                e.preventDefault();
                $(this).click();
            }
        });
    });
}

/*-----------------------------------------------------*\
    @DRAG-DROP
    Keyboard support for list page drag/drop (a.handle) elements.
    Status messages and screen reader updates specifically for the screen reader
\*-----------------------------------------------------*/

/* Function that returns the accessibility context for a given list item based on the list item index (1+) */
function getListItemA11yContext(oneBasedIndex) {
    // Support both li (card view) and tr (list view) elements
    let listItem = $('#divAllItems li:nth-child(' + oneBasedIndex + '), #divAllItems tr:nth-child(' + oneBasedIndex + ')');
    if (listItem.length < 1) {
        console.log('> getListItemA11yContext() nthIndex=' + oneBasedIndex + ' not found.');
        return;
    }
    let a11yContainer = $(listItem).find('div.listing-element[role="group"]');
    if (a11yContainer.length < 1) {
        a11yContainer = $(listItem).find('div.row[role="group"]');
    }
    if (a11yContainer.length === 1) {
        let accLabel = a11yContainer.attr('aria-label');
        if (!isEmpty(accLabel)) {
            return accLabel;
        }
    }
    return '';
}

/* Function that checks the page context for drag/drop functionality and if found, registers associated event handlers for keyboard navigation */
function setupA11yDragDropKeyboardSupport() {
    // Support both ul (card view) and tbody (list view) containers
    if ($('ul#divAllItems').length !== 1 && $('tbody#divAllItems').length !== 1) {
        return; // not supported
    }
    let dragDropHandles = $('a.handle');
    for (var i=0; i<dragDropHandles.length; i++) {
        let dragDropHandle = dragDropHandles[i];

        // drag/drop element not currently setup for keyboard support (ex. page load, load more or filter change)
        if (dragDropHandle.getAttribute('data-a11y-drag-drop') !== 'true') {            
            dragDropHandle.setAttribute('data-a11y-drag-drop', 'true');
            dragDropHandle.setAttribute('role', 'application');
            dragDropHandle.setAttribute('aria-label', 'Drag and drop. Press enter to activate.');
            dragDropHandle.setAttribute('onkeydown', 'handleA11yKeyboardDragDrop(event);');
        }

        // add event listener on body for mouse-down (will cancel an active drag/drop operation)
        if ($('body.a11y-drag-drop').length < 1) {
            document.body.classList.add('a11y-drag-drop');
            document.body.removeEventListener('mousedown', handleA11yKeyboardDragDropMouseDown);
            document.body.addEventListener('mousedown', handleA11yKeyboardDragDropMouseDown);
        }
    }
    if (dragDropHandles.length > 0) {
        // Empty message to make sure the announcemeent region is defined (otherwise the first message will not be read by the screen reader)
        a11yAnnounceToScreenReader('');
    }
}

/* Function that returns the DOM link element for the active drag/drop operation (the active handle) */
function getA11yDragDropHandleFocusLink() {
    let handleFocusLink = $('[data-a11y-drag-drop-focus="true"]');
    if (handleFocusLink.length === 1) {
        return handleFocusLink[0];
    }
}

/* Function that returns the DOM li or tr element container for the active drag/drop operation (li/tr container for the active handle) */
function getA11yDragDropHandleFocusLinkLi(handleLink) {
  // Support both li (card view) and tr (list view) elements
  let handleClosestItem = $(handleLink).closest('li, tr');
  if (handleClosestItem.length !== 1) {
    console.error('> DRAG/DROP handleClosestItem (li or tr) not found');
    handleA11yDragDropFailure();
    cancelA11yDragDrop();
    return;
  }
  return handleClosestItem[0];
}

/* Function that returns true if there is an active drag/drop operation initiated by the keyboard */
function isA11yDragDropActive(handleFocusLink) {
    if (!handleFocusLink) {
        handleFocusLink = getA11yDragDropHandleFocusLink();
    }
    if (handleFocusLink && handleFocusLink.getAttribute('data-a11y-drag-drop-focus') === 'true') {
        return true;
    }
    return false;
}

/* Function that removes state attributes or classes used during a keyboard drag/drop operation (on save or cancel) */
function a11yDragDropPickup(handleLink, dragDropIndex) {
    // this element will retain focus during the drag/drop operation (store the original index to restore on cancel)
    handleLink.setAttribute('data-a11y-drag-drop-focus', 'true');
    handleLink.setAttribute('data-a11y-drag-drop-original-index', dragDropIndex);
    
    let totalItems = getA11yDragDropTotalItems();
    let itemPosition = getA11yDragDropItemPosition(handleLink);
    let a11yMessage = 'Drag and drop. Pickup from position ' + itemPosition + ' of  ' + totalItems + '.';
    let a11yContext = getListItemA11yContext(itemPosition);
    if (!isEmpty(a11yContext)) {
        a11yMessage += ' You picked up ' + a11yContext;
    }
    a11yAnnounceToScreenReader(a11yMessage);
}

/* Function that removes state attributes or classes used during a keyboard drag/drop operation (on save or cancel) */
function cleanupAfterA11yDragDrop() {
    let handleFocusLink = getA11yDragDropHandleFocusLink();
    if (handleFocusLink) {
        handleFocusLink.focus();
    } else {
        console.error('> DRAG/DROP handleFocusLink not found');
        handleA11yDragDropFailure();
        setFocusToContent();
    }
    $('[data-a11y-drag-drop-focus]').removeAttr('data-a11y-drag-drop-focus');
    $('[data-a11y-drag-drop-original-index]').removeAttr('data-a11y-drag-drop-original-index');
}

/* Function that cancels a currently active drag/drop operation initiated by the keyboard */
function cancelA11yDragDrop() {
    let handleFocusLink = getA11yDragDropHandleFocusLink();
    if (!handleFocusLink) {
        cleanupAfterA11yDragDrop();
        return;
    }
    let handleLi = getA11yDragDropHandleFocusLinkLi(handleFocusLink);
    if (!handleLi) {
        return;
    }

    let originalIndexText = handleFocusLink.getAttribute('data-a11y-drag-drop-original-index');
    if (!originalIndexText) {
        console.error('> DRAG/DROP originalIndexText not found');
        cleanupAfterA11yDragDrop();
        return;
    }
    let originalIndex = parseInt(originalIndexText);
    let distanceFromOriginalPosition = $(handleLi).index() - originalIndex;

    // Return the list item to it's original position
    while (distanceFromOriginalPosition > 0) {
        handleA11yKeyboardDragDropArrowUpDown(handleFocusLink, true, false); // move up
        distanceFromOriginalPosition--;
    }
    while (distanceFromOriginalPosition < 0) {
        handleA11yKeyboardDragDropArrowUpDown(handleFocusLink, false, false); // move down
        distanceFromOriginalPosition++;
    }

    cleanupAfterA11yDragDrop();
    a11yAnnounceToScreenReader('Drag and drop. Operation cancelled.');
}

/* Function that returns the total number of items in the drag/drop list */
function getA11yDragDropTotalItems() {
    // Support both li (card view) and tr (list view) elements
    return $('#divAllItems > li:not(.unsortable_item), #divAllItems > tr:not(.unsortable_item)').length;
}
/* Function that returns the position of the provided handle link within the UL or TBODY */
function getA11yDragDropItemPosition(elem) {
    // Support both li (card view) and tr (list view) elements
    if (elem && elem.tagName !== 'LI' && elem.tagName !== 'TR') {
        elem = getA11yDragDropHandleFocusLinkLi(elem);
    }
    if (!elem) {
        console.error('> DRAG/DROP elem is empty');
        return;
    }
    return $(elem).prevAll(':not(.unsortable_item)').length + 1;
}

/* Function that saves the list order after completing the drag/drop operation by dropping the handle link */
function saveA11yDragDrop(handleLink) {
    let handleLi = getA11yDragDropHandleFocusLinkLi(handleLink);
    if (!handleLi) {
        return;
    }

    let sortableContainer = $(handleLi).parents('.ui-sortable');
    if (sortableContainer.length < 1) {
        console.error('> DRAG/DROP sortableContainer not found');
        handleA11yDragDropFailure();
        cleanupAfterA11yDragDrop();
        return;
    }

    let totalItems = getA11yDragDropTotalItems();
    let itemPosition = getA11yDragDropItemPosition(handleLi);
    a11yAnnounceToScreenReader('Drag and drop. Drop into position ' + itemPosition + ' of ' + totalItems + '.');
    if (a11yDragDropSaveOrdering) {
        a11yDragDropSaveOrdering(handleLi);
    }
    cleanupAfterA11yDragDrop();
}

/* Function that cancels the drag/drop operation if the mouse is clicked */
function handleA11yKeyboardDragDropMouseDown() {
    if (isA11yDragDropActive()) {
        cancelA11yDragDrop();
    }
}

/* Function that generically notifies the user of an error in the drag/drop operation using a screen reader announcement */
function handleA11yDragDropFailure() {
    a11yAnnounceToScreenReader('Drag and drop. Something went wrong, please wait a few minutes and try again.');
}

/**
 * Handles the arrow up or down key during an active drag/drop operation to move the current item
 * one position in the list (if possible).
 *
 * @param {HTMLElement} handleLink - The anchor (.handle) element that triggered the keydown event.
 * @param {boolean} isUp - `true` if the Arrow Up key was pressed; `false` if Arrow Down.
 * @param {boolean} [announceToScreenReader=true] - Whether to announce the change to the screen reader (optional, defaults to `true`).
 */
function handleA11yKeyboardDragDropArrowUpDown(handleLink, isUp, announceToScreenReader = true) {
  if (!(isUp === true || isUp === false)) {
    console.error('> DRAG/DROP isUp must be true or false');
    handleA11yDragDropFailure();
    return;
  }
  if (!handleLink) {
    console.error('> DRAG/DROP handleLink is empty');
    handleA11yDragDropFailure();
    return;
  }
  let handleLi = getA11yDragDropHandleFocusLinkLi(handleLink);
  if (!handleLi) {
    return;
  }

  let sortableContainer = $(handleLi).parents('.ui-sortable');
  if (sortableContainer.length < 1) {
    console.error('> DRAG/DROP sortableContainer not found');
    handleA11yDragDropFailure();
    return;
  }
  try {
    let siblingLi = isUp ? $(handleLi).prev() : $(handleLi).next();
    if(siblingLi.length === 0){
      a11yAnnounceToScreenReader(`Drag and drop. You are at the ${isUp ? 'top' : 'bottom'} of the list.`);
      return;
    }
    if(isUp){
      $(handleLi).insertBefore(siblingLi);
    } else {
      $(handleLi).insertAfter(siblingLi);
    }
    sortableContainer.sortable('refresh');

    let handleFocusLink = getA11yDragDropHandleFocusLink();
    if (!handleFocusLink) {
      console.error('> DRAG/DROP handleFocusLink not found');
      handleA11yDragDropFailure();
      return;
    }
    handleFocusLink.focus();
    if (announceToScreenReader) {
      let itemPosition = getA11yDragDropItemPosition(handleFocusLink);
      let message = `Drag and drop. Move ${isUp ? 'up' : 'down'} to position ${itemPosition} of ${getA11yDragDropTotalItems()}.`;
      let a11yContext = getListItemA11yContext(itemPosition + (isUp ? 1 : -1));
      if (!isEmpty(a11yContext)) {
        message += " " + (isUp ? 'Above ' : 'Below ') + a11yContext;
      }
      a11yAnnounceToScreenReader(message);
    }
  } catch(e) {
    console.error('> DRAG/DROP Error e.message=' + e.message);
    handleA11yDragDropFailure();
  }
}

/* Function that handles keyboard support for drag/drop functionality. Handles the following keys when in drag/drop mode:
 * 
 * Enter      : Pick up (from it's current location) or drop an item (at it's current location)
 * Space      : Same as Enter
 * Escape     : Cancel the drag/drop operation
 * Arrow-Up   : Move the item up one position if possible (if at the top of the list do nothing)
 * Arrow-Down : Move the item down on position if possible (if at the bottom of the list do nothing)
 * 
 * The Screen reader should announce changes during the drag/drop operation:
 * 
 * Pick up    : Drag and drop. You have selected [item title] in position [N of M].
 * Drop       : Dropping [item title] into position [N of M]
 * Arrow up   : Moving [item title] to position [N of M]
 * Arrow down : Moving [item title] to position [N of M]
 * 
 * Top/Bottom : You're at the [top/bottom] of the list
 * Escape     : Drag and drop has been cancelled.
 * Mouse down : Drag and drop has been cancelled.
 * 
 * We'll call a11yAnnounceToScreenReader('status') during the drag/drop operation as status changes
 * 
 */
function handleA11yKeyboardDragDrop(event)
{
    if (!isAccKeyboardMode()) {
        return; // nothing to do
    }
    let elem = document.activeElement;
    if (isEmpty(elem)) {
        console.error('> DRAG/DROP Error elem is empty (no active element found)');
        return;
    }
    let isElemDragDropActive = isA11yDragDropActive(elem);
    if (isA11yDragDropActive() && !isElemDragDropActive) {
        console.error('> DRAG/DROP is active but not for this element (unexpected condition)');
        handleA11yDragDropFailure();
        cleanupAfterA11yDragDrop();
        return;
    }
    switch (event.key) {
        case "ArrowUp":
            if (isElemDragDropActive) {
                // move item one position up the list
                event.preventDefault();
                event.stopPropagation();
                handleA11yKeyboardDragDropArrowUpDown(elem, true);
            }
            break;
        case "ArrowDown":
            if (isElemDragDropActive) {
                // move item one position down the list
                event.preventDefault();
                event.stopPropagation();
                handleA11yKeyboardDragDropArrowUpDown(elem, false);
            }
            break;
        case "Enter":
        case "Space":            
            event.preventDefault();
            event.stopPropagation();
            if (!isElemDragDropActive) {
                // Pick-up from current position
                let handleLi = getA11yDragDropHandleFocusLinkLi(elem);
                if (!handleLi) {
                    return;
                }
                a11yDragDropPickup(elem, $(handleLi).index());
            } else {
                // Drop item (into new location at current position)
                saveA11yDragDrop(elem);
            }
            break;
        case "Escape":
            // Cancel the drag/drop operation
            if (isElemDragDropActive) {
                event.preventDefault();
                event.stopPropagation();
                cancelA11yDragDrop();
            }
            break;
        case "Tab":
            if (isElemDragDropActive) {
                if (event.ctrlKey || event.altKey || event.metaKey) {
                    // do nothing (we only track tab/shift-tab here)
                    return;
                }
                event.preventDefault();
                event.stopPropagation();
                a11yAnnounceToScreenReader('Drag and drop. Press escape or click the mouse to cancel the current drag and drop operation.');
            }
            break;
    }
}

/*-----------------------------------------------------*\
    @END
\*-----------------------------------------------------*/
