function initSlickEvents() {
    $('.custom-page-slider.slick-initialized').slick('unslick');
    $.each($('.custom-page-slider'), function(index, el){
        var elid = $(el)[0].id;
        var elDataset = $(el)[0].dataset;
        var elParentElement = $(el).parent()[0];
        var $parentLineElement = $(el).parents("[data-page-line-element]");
        var slickParams = {
            slidesToShow: 4,
            slidesToScroll: 4,
            adaptiveHeight: true,
            infinite: false,
            responsive: getSliderBreakpoints($parentLineElement, elDataset),
            prevArrow: '<h3 id="h2_prev_' + elid + '" role="button" tabindex="0" aria-label="previous" aria-description="Click to get the previous slides then tab forward." class="slider-arrow-previous header-cg--h2"><span id="span_prev_' + elid + '" class="mdi mdi-chevron-left"></span><span class="visually-hidden" aria-label="previous">previous</span></h3>',
            nextArrow: '<h3 id="h2_next_' + elid + '" role="button" tabindex="0" aria-label="next" aria-description="Click to get the next slides then shift-tab back to the slide deck." class="slider-arrow-next header-cg--h2"><span id="span_next_' + elid + '" class="mdi mdi-chevron-right"></span><span class="visually-hidden" aria-label="next">next</span></h3>'
        }
        if(typeof elDataset !== "undefined"){
            if(typeof elDataset.sliderVariableWidth !== "undefined" && elDataset.sliderVariableWidth == "1"){
                slickParams.variableWidth = true;
            }
            if(typeof elDataset.slidesToShow !== "undefined"){
                slickParams.slidesToShow = parseInt(elDataset.slidesToShow, 10);
            }
            slickParams.slidesToScroll = typeof elDataset.slidesToScroll !== "undefined" ? parseInt(elDataset.slidesToScroll, 10) : slickParams.slidesToShow;
        }
        
        $(el).not('.slick-initialized').slick(slickParams);

        if (!isEmpty(elParentElement.id)) {
            var divWrapperId = elParentElement.id;
            var h2SlideshowPreviousId = 'h2_prev_' + elid;
            var h2SlideshowNextId = 'h2_next_' + elid;
            var spanSlideshowPreviousId = 'span_prev_' + elid;
            var spanSlideshowNextId = 'span_next_' + elid;
            setupKeyboardNavigationForEventCarouselSlideshow(divWrapperId, h2SlideshowPreviousId, h2SlideshowNextId, spanSlideshowPreviousId, spanSlideshowNextId);
        }
    });
}

function getSliderBreakpoints($container, elDataset = {}){
    var bootstrapWidth = parseInt($container.attr("data-page-line-element-size"), 10);
    if (!isNaN(bootstrapWidth)){
        if(bootstrapWidth <= 4){
            var br3200show = typeof elDataset['slidesToShow-4-3200'] !== "undefined" ? parseInt(elDataset['slidesToShow-4-3200']) : 1;
            return [
                {
                    breakpoint: 3200,
                    settings: {
                        slidesToShow: br3200show,
                        slidesToScroll: typeof elDataset['slidesToScroll-4-3200'] !== "undefined" ? parseInt(elDataset['slidesToScroll-4-3200']) : br3200show
                    }
                }
            ]
        } else if(bootstrapWidth <= 6){
            var br3200show = typeof elDataset['slidesToShow-6-3200'] !== "undefined" ? parseInt(elDataset['slidesToShow-6-3200']) : 2;
            var br1600show = typeof elDataset['slidesToShow-6-1600'] !== "undefined" ? parseInt(elDataset['slidesToShow-6-1600']) : 1;
            return [
                {
                    breakpoint: 3200,
                    settings: {
                        slidesToShow: br3200show,
                        slidesToScroll: typeof elDataset['slidesToScroll-6-3200'] !== "undefined" ? parseInt(elDataset['slidesToScroll-6-3200']) : br3200show
                    }
                },
                {
                    breakpoint: 1600,
                    settings: {
                        slidesToShow: br1600show,
                        slidesToScroll: typeof elDataset['slidesToScroll-6-1600'] !== "undefined" ? parseInt(elDataset['slidesToScroll-6-1600']) : br1600show
                    }
                }
            ]
        } else if(bootstrapWidth <= 9){
            var br2208show = typeof elDataset['slidesToShow-9-2208'] !== "undefined" ? parseInt(elDataset['slidesToShow-9-2208']) : 3;
            var br1600show = typeof elDataset['slidesToShow-9-1600'] !== "undefined" ? parseInt(elDataset['slidesToShow-9-1600']) : 2;
            var br992show = typeof elDataset['slidesToShow-9-992'] !== "undefined" ? parseInt(elDataset['slidesToShow-9-992']) : 1;
            console.log(typeof elDataset['slidesToShow-9-2208'], "hello");
            return [
                {
                    breakpoint: 2208,
                    settings: {
                        slidesToShow: br2208show,
                        slidesToScroll: typeof elDataset['slidesToScroll-9-2208'] !== "undefined" ? parseInt(elDataset['slidesToScroll-9-2208']) : br2208show
                    }
                },
                {
                    breakpoint: 1600,
                    settings: {
                        slidesToShow: br1600show,
                        slidesToScroll: typeof elDataset['slidesToScroll-9-1600'] !== "undefined" ? parseInt(elDataset['slidesToScroll-9-1600']) : br1600show
                    }
                },
                {
                    breakpoint: 992,
                    settings: {
                        slidesToShow: br992show,
                        slidesToScroll: typeof elDataset['slidesToScroll-9-992'] !== "undefined" ? parseInt(elDataset['slidesToScroll-9-992']) : br992show
                    }
                }
            ]
        }
    }

    var br770show = typeof elDataset['slidesToShow-770'] !== "undefined" ? parseInt(elDataset['slidesToShow-770']) : 1;
    var br992show = typeof elDataset['slidesToShow-992'] !== "undefined" ? parseInt(elDataset['slidesToShow-992']) : 2;
    var br1600show = typeof elDataset['slidesToShow-1600'] !== "undefined" ? parseInt(elDataset['slidesToShow-1600']) : 3;
    return [
        {
            breakpoint: 770,
            settings: {
                slidesToShow: br770show,
                slidesToScroll: typeof elDataset['slidesToScroll-770'] !== "undefined" ? parseInt(elDataset['slidesToScroll-770']) : br770show
            }
        },
        {
            breakpoint: 992,
            settings: {
                slidesToShow: br992show,
                slidesToScroll: typeof elDataset['slidesToScroll-992'] !== "undefined" ? parseInt(elDataset['slidesToScroll-992']) : br992show
            }
        },
        {
            breakpoint: 1600,
            settings: {
                slidesToShow: br1600show,
                slidesToScroll: typeof elDataset['slidesToScroll-1600'] !== "undefined" ? parseInt(elDataset['slidesToScroll-1600']) : br1600show
            }
        }
    ]
}

function initPage(){
    initSlickEvents();
    hideLoading();
    clampText('text-clamp');
}

function showLoading(){
    var $pageCont = $("#page-cont");
    $(".page-wrapper").css("visibility", "hidden");
    $pageCont.children(".loader").remove();
    $pageCont.append('<p class="loader text-center" style="position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);"><img alt="Loading" src="/images/loader-big.gif"><br> Loading page...</p>');
}

function hideLoading(){
    $(".page-wrapper").css("visibility", "inherit");
    $("#page-cont").find(".loader").remove();
}

$(document).ready(function(){
    initPage();
});

// We use onLoad event to wait the loading of images so we'll have the correct height of elements
$(window).on("load", function() {
    initPage();
});

$('.dropdown-countries a').on('click', function (e) {
    e.preventDefault();
    var country = $(this).data("country");
    $(this).parents("[data-line-id]").siblings().remove();
    $(this).parents("[data-line-id]").parent().append('<li><p class="loader text-center" style="position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);"><img alt="Loading" src="/images/loader-big.gif"><br> Loading page...</p></li>');

    var url = new URL(window.location.origin + "/page?ax=1");
    url.searchParams.set('country', country);
    url.searchParams.set('id', $(".page-wrapper").attr("data-page-id"));
    getContentNew("#page-cont", url.href, function (a,b) {console.log(a, b)}, function (a,b){console.log(a, b)});
    var countryText = $(this).find(".country").text();
    $(this).parent().parent().parent().find("button .country").text(countryText);
});

// Home Page JS Logic after Group Pinned
$("[id^='group-card__'] .pin-btn").bind("click", function(){
    var _element = $(this).parent().parent();
    if($(this).find(".btn.btn-cg--group").length > 0) {
        $(_element).fadeOut(300, function() {
            $("#groups-cont").prepend($(_element));
            $(_element).fadeIn(500);
        });
    }
    else {
        $(_element).fadeOut(300, function() {
            $("#groups-cont").append($(_element));
            $(_element).fadeIn(500);
        });
    }
});
