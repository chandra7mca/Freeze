;(function( $, window, document, undefined ){
	
	if (!$.plugins) {
        $.plugins = {};
    }
	
	$.freeze = function ( el, options ) {
		
        // To avoid scope issues, use 'base' instead of 'this'
        // to reference this class from internal events and functions.
        var base = this;
        
        // this will hold the merged default, and user-provided options
        // plugin's properties will be available through this object like:
        // plugin.settings.propertyName from inside the plugin or
        // element.data('pluginName').settings.propertyName from outside the plugin, 
        // where "element" is the element the plugin is attached to;
        base.options = {};
        
        // Access to jQuery and DOM versions of element
        base.$el = $(el);
        base.el = el;

        // Access to Container element as Jquery Object
        var $persistAreaContainer = base.$el;
        var containerTopPos = $persistAreaContainer.offset().top;

        // Add a reverse reference to the DOM object
        //base.$el.data( "plugins.freeze" , base ); // added while creating the plugin

        // the "constructor" method that gets called when the object is created
        base.init = function () {
        	console.log("Start of Init");
          //  base.myFunctionParam = myFunctionParam;

            /** Extend Default options with those Provided
    		 *  Note that the first argument the first argument to extend is an empty 
    		 *  object - this is to keep from overriding our "defaults" object 
    		 */
            base.options = $.extend({}, $.freeze.defaultOptions, options);

            // Put your initialization code here
            
            var activeHeadersArray = [];            

        	$persistAreaContainer.data("activeHeadersArray", activeHeadersArray);
        	
        	// Process Freezing Headers on Scroll
        	if( $persistAreaContainer.data('hasFixedHeader') === 'true' ) {
        		var $headerRow = $("."+base.options.freezeElemClass, $persistAreaContainer).first();
        		var $floatingHeader = $("."+base.options.frozenElemClass, $persistAreaContainer).first();

        		base.initFloatingHeader($headerRow, $floatingHeader, true );

        		var $headerRowChildren = $headerRow.children();
        		
        		$persistAreaContainer.data('totalSortColumns', $headerRowChildren.length );

        		$headerRowChildren.each(function(index,val){
        			var sortOrder = $persistAreaContainer.data('column-'+index+'-SortOrder');
        			if( sortOrder == null || sortOrder == undefined || sortOrder === '' || sortOrder === 'unsorted' )
        				sortOrder = 'unsorted';

        			$(this).removeClass('unsorted').removeClass('asc').removeClass('desc').addClass(sortOrder);
        			$floatingHeader.children().eq($(this).index()).removeClass('unsorted').removeClass('asc').removeClass('desc').addClass(sortOrder);
        		});
        		
        	}
        	
        	// Clone Fixed Headers from existing Headers, if floating Headers are not present
        	$("."+base.options.freezeAreaClass, $persistAreaContainer).each(function() {
        		var $persistArea = $(this);
        		var $headerRow = $("."+base.options.freezeElemClass, $persistArea).first();
        		var $floatingHeader = $("."+base.options.frozenElemClass, $persistArea).first();

        		base.initFloatingHeader($headerRow, $floatingHeader, false );

        		// Specific for IE7
        		if( $headerRow.hasClass('hide') )
        			$headerRow.removeClass('hide');
        		   
        		// Position Calculator
        		//var $persistArea = $(this);

        		var $persistAreaElemCalc = new $.PositionCalculator({
        			item: $persistArea,
        			boundary: $persistAreaContainer,
        			stick:false
        		});

        		var result = $persistAreaElemCalc.calculate();
        		$persistArea.data('initialTop', result.distance.top);
        	});
        	
        	console.log("End of Init");
        };

        base.initFloatingHeader = function($headerRow, $floatingHeader, isFixedContainerHeader ) {
        	console.log("Start of InitFloating Header");
        	if( $floatingHeader.length <= 0 ) {// > 0, if needed to clone the persist-header as floatingHeader
    			$floatingHeader = $headerRow.clone(true); // If floating header was not provided by default
    			$headerRow.after( $floatingHeader ); // Append as next element if cloned
    			$floatingHeader.addClass(base.options.frozenElemClass).removeClass(base.options.freezeElemClass);
    		}
    		
    		$headerRow.data('clonedCopy', $floatingHeader);
    		
    		if( isFixedContainerHeader === 'true' )
    			$floatingHeader.data('clonedCopy', $headerRow).addClass('fixedContainerHeader').addClass(base.options.frozenElemClass);
    		else
    			$floatingHeader.data('clonedCopy', $headerRow).addClass(base.options.frozenElemClass);
    		
    		console.log("End of InitFloatingHeader");
        };
        
        base.onScroll = function(e){
        	console.log("Start of OnScroll");
    		e.preventDefault();

    		$persistAreaContainer.data("activeHeadersArray", []);
    		
    		var elementCount = 0;
    		var $persisAreaElems = $('.'+base.options.freezeAreaClass+':visible', $persistAreaContainer) ;
    		var persistAreaElemsToProcess = [];
    		var totalPersistAreaElems = $persisAreaElems.length;
    		var persistAreaToConsider = 5;

    		var $persistArea = null;
    		var persistAreaHeight = null;
    		var persistAreaTopPos = null;
    		
    		var persistAreaOffset = null;
    		
    		for( var i = 0 ; i < totalPersistAreaElems ; i++ ) {
    			$persistArea = $persisAreaElems.eq(i);
    		
    			persistAreaHeight = $persistArea.outerHeight();
    			persistAreaTopPos = $persistArea.offset().top;
    			
    			persistAreaOffset = persistAreaTopPos + persistAreaHeight;
    			if( elementCount > persistAreaToConsider )
    				break;
    			else if( persistAreaOffset  >= containerTopPos ) {
    				elementCount++;
    				persistAreaElemsToProcess.push($persistArea);
    			}
    		}

    		var $persistAreaElemCalc = null;
    		var $persistHeader  = null;
    		var $persistAreaEnd = null;
    		
    		var result = null;
    		var initialTop = null;
    		
    		var positionFloatingHeaderFunc = null;
    		var processInviewFunc = null;

    		var persistAreaHeight = null;
    		
    		$(persistAreaElemsToProcess).each(function(){	
    			$persistArea = $(this);

    			$persistAreaElemCalc = new $.PositionCalculator({
    				item: $persistArea,
    				boundary: $persistAreaContainer,
    				stick:false
    			});
    			$persistHeader  = $("."+base.options.freezeElemClass, $persistArea).first();
    			$persistAreaEnd = $("."+base.options.freezeAreaEndClass, $persistArea ).last();
    			
    			result = $persistAreaElemCalc.calculate();
    			initialTop = $persistArea.data('initialTop');
    			
    			positionFloatingHeaderFunc = base.positionFloatingHeader;
    			processInviewFunc = base.processInView;

    			persistAreaTopPos = result.distance.top;
    			persistAreaHeight = $persistArea.outerHeight();
    			
    			$persistArea.data('initialTop', persistAreaTopPos);
    			
    			// Process Floating Header for only Persist-Areas which are in view
    			if( persistAreaTopPos >= 0 && ( persistAreaTopPos <= ( initialTop + persistAreaHeight ) ) ) {
    				if( ! $persistHeader.hasClass('floatingHeaderProcessed') )
    					positionFloatingHeaderFunc( $persistArea,$persistHeader);

    				processInviewFunc($persistArea,$persistHeader,$persistAreaEnd,$persistAreaContainer);
    			} else
    				$persistHeader.next().css({ "display" : "none", "top" : "0px" });
    		});
    		console.log("End of OnScroll");
        };
        
        
        base.processInView = function($persistArea,$persistHeader,$persistAreaEnd,$persistAreaContainer) {
        	console.log("start of ProcessInView");

        	var persistHeaderElemCalc = new $.PositionCalculator({
        		  item: $persistHeader,
        		  boundary: $persistAreaContainer,
        		  stick:false
        	  });

        	var headerResult = persistHeaderElemCalc.calculate();
        	var elemHeight = $persistHeader.outerHeight();	// Header Height
        	var floatingHeader = $("."+base.options.frozenElemClass, $persistArea).first();

        	var previousHeadersTotalHeight = 0;
        	
        	var activeHeadersArray = $persistAreaContainer.data("activeHeadersArray");
        	var totActiveHeaders = activeHeadersArray.length;
        	/*
        	 var heightOfContainerFixedFloatingHeader = 0;
        	
        	// If it is first and container level fixed Header
        	if( $persistAreaContainer.data('hasFixedHeader') === 'true' ) {
        		
        		if( floatingHeader.hasClass('fixedContainerHeader') ) // i.e., If Current Floating Header is for entire Container
        			heightOfContainerFixedFloatingHeader = 0; // Do not consider the height for the container level fixed floating header
        		else {
        			var $containerLevelFixedHeader = $('.floatingHeader', $persistAreaContainer).first();
        			heightOfContainerFixedFloatingHeader = $containerLevelFixedHeader.outerHeight(); // Get the container level fixed floating header height so as other fixed headers will 
        																				// be positioned based on it
        		}
        		
        		//previousHeadersTotalHeight += heightOfContainerFixedFloatingHeader;
        	}
        	*/
        	for( var i = 0 ; i < totActiveHeaders ; i++ ) {
        		previousHeadersTotalHeight += activeHeadersArray[i];
        	}

        	var headerTopPos = headerResult.distance.top;

        	// For Vertical Scrolling
        	if( ( headerTopPos + previousHeadersTotalHeight) > 0 ) { // Element is out of View
        		var persistAreaEndElemCalc = new $.PositionCalculator({
        			item: $persistAreaEnd,
        			boundary: $persistAreaContainer,
        			stick:false
        		});

            	var persistsAreaEndResult = persistAreaEndElemCalc.calculate();
            	var persistAreaEndTopPos = persistsAreaEndResult.distance.top;

            	if( ( ( persistAreaEndTopPos + (2*elemHeight)) > 0 ) && ( persistAreaEndTopPos <= headerTopPos ) ) {
            		floatingHeader.css({ "display" : "none", "top" : "0px" });
            		//activeHeadersArray.pop();
            	}
            	else {
        			floatingHeader.css({ "display" : $persistHeader.css('display'), "top" : ($persistAreaContainer.scrollTop()+ previousHeadersTotalHeight )+"px" });
        			activeHeadersArray.push(elemHeight);
            	}
              } else {
            	  floatingHeader.css({ "display" : "none", "top" : "0px" });
            	  activeHeadersArray.pop();
              }
        	
        	
        	$persistAreaContainer.data("activeHeadersArray",activeHeadersArray);
        	console.log("End of ProcessInView");
        };

        base.positionFloatingHeader = function( $persistArea, $headerRow) {
        	console.log("Start of PositionFloatingHeader");
        	
        	var floatingHeader = $headerRow.next();
        	/*
        		headerRow.clone(); -- if floating header was not provided by default
        		if( $('.floatingHeader', this).first().length > 1 ) // > 0, if needed to clone the persist-header as floatingHeader
        			return;
        	 	
        		headerRow.after(floatingHeader); -- Append as next element if cloned
        		$headerRow.data('clonedCopy', floatingHeader);
        		floatingHeader.data('clonedCopy', $headerRow);
        		
        		floatingHeader.addClass("floatingHeader").removeClass('persist-header'); //.data("floatingHeaderIndex",floatingHeaderCounter++);
        	 */
               
        	var bgColor = $headerRow.css("backgroundColor");
            
        	//var floatingHeaderHtmlContent = "<div style='width: "+$headerRow.width()+"px;'>"+ floatingHeader.html() + "</div>";
        	
        	//floatingHeader.html(floatingHeaderHtmlContent).css({ "background" : rowBgColor });
        	floatingHeader.css({ "background" : bgColor });
        	var floatingHeaderBuilt = true;

        	var $sourceChildren = $headerRow.children();
        	var $targetChildren = floatingHeader.children();

        	while( $sourceChildren.length > 0  ){

        		floatingHeaderBuilt = base.applyElemWidths( $sourceChildren, $targetChildren , bgColor );
        		if( floatingHeaderBuilt == false )
        			break;
        		
        		$sourceChildren = $sourceChildren.children();
        		$targetChildren = $targetChildren.children();
        	}
        	if( floatingHeaderBuilt == true )
        		$headerRow.addClass('floatingHeaderProcessed');
        	
        	console.log("End of PositionFloatingHeader");
        };
        
        base.applyElemWidths = function( $sourceChildren, $targetChildren , bgColor ) {
        	console.log("Start of ApplyElemWidths");

        	var i = 0;
        	var floatingHeaderBuilt =  true;
        	var totalChildren = $sourceChildren.length;
        	var $srcElem = null;
        	var $targetElem = null;
        	var elemWidth = null;
        	var htmlContent = null;
        	var srcTagName = null;
        	
        	while( i < totalChildren ) {
        		$srcElem =  $sourceChildren.eq(i);
        		$targetElem = $targetChildren.eq(i);
        		
        		elemWidth = $srcElem.width();
        		
        		if( $.browser.webkit || $.browser.msie)
        			elemWidth++;

        		if( elemWidth < 2 )	// If '.persist-header' is not visible, then cell width will be obtained as either 0 or 1 in various browsers
        			return floatingHeaderBuilt = false;

        		htmlContent = "";
        		srcTagName = $srcElem.prop("tagName");
        		
        		if( srcTagName == null || srcTagName == undefined )
        			srcTagName = "";
        		else
        			srcTagName = srcTagName.toLowerCase();

        		if( srcTagName === 'td' || srcTagName === 'th' )
        			htmlContent = "<div style='width: "+elemWidth+"px;'>"+ $targetElem.html() + "</div>";
        		else
        			htmlContent = $targetElem.html();
        		
        		$targetElem.html(htmlContent); //.css({ "background" : bgColor });
        		
        		//console.log("<br>--> "+$sourceChildren.eq(i).html());
        		i++;
        	}
        	
        	console.log("End of ApplyElemWidths");
        	return floatingHeaderBuilt;
        };

        base.debug = function() {
        	console.log("Debug");
        };

        base.complete = function() {
        	console.log("Complete");
        };

        base.bindScrollEvent = function() {
        	$persistAreaContainer.on('scroll', base.onScroll);
        };
        
        base.updateContainerFloatingHeader = function() {
        	var $persistArea = $(".persist-area", $persistAreaContainer).first();
        	var $headerRow = $(".persist-header", $persistArea).first();
        	
        	base.positionFloatingHeader($persistArea, $headerRow);	
        };

        base.reset = function() {
        	$persistAreaContainer.off();
        	if( $.browser.msie ) {
        		$persistAreaContainer.find('.floatingHeader').first().css({ "display" : "none" });
        		$nestedTableElems.css({ "display" : "none" });
        	}
        		
        	$('.persist-area',$persistAreaContainer).each(function(){
        		$(this).unbind();
        	});
        };
        
        base.init();
        base.bindScrollEvent();
    };

    $.freeze.defaultOptions = {
    		freezeAreaClass: 'persist-area',
    		freezeAreaEndClass: 'persist-area-end',
    		freezeElemClass : 'persist-header',
    		frozenElemClass : 'floatingHeader',
    		containerHasFixedHeader : 'true'
    };

    // add the plugin to the jQuery.fn object
    $.fn.freeze = function(options) {

        // iterate through the DOM elements we are attaching the plugin to
        return this.each(function() {
            // if plugin has not already been attached to the element
            if (undefined == $(this).data('pluginName')) {

                // create a new instance of the plugin
                // pass the DOM element and the user-provided options as arguments
                var plugin = new $.freeze(this, options);

                // in the jQuery version of the element
                // store a reference to the plugin object
                // you can later access the plugin and its methods and properties like
                // element.data('pluginName').publicMethod(arg1, arg2, ... argn) or
                // element.data('pluginName').settings.propertyName
                $(this).data('freeze', plugin);
            }

        });
    };

})( jQuery );
