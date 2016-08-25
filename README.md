# ember-code-sample

## Action for Login

> This is the action called when the Login button is clicked.


```
login: function() {
  // get the app controller
  var app = this.get("applicationController");
  var u = this.get("username");
  var p = this.get("password");
     
  // if user and pass input exist
  if (u && u.length > 1 && p && p.length > 1) {
      var that = this;
      // post to api login
      $(".spinner").show();
      Ember.$.ajax({
          url: App.HostFullPath + 'login/',
          type: 'POST',
          data: {
              "username": u,
              "password": p
          },
          accepts: 'application/json',

          // if it works save the user in application controller user property
          success: function(data) {
              try {
                  // var user = JSON.parse(data.user);
                  //no need to parse object as we are not sending user as string
                  var user = data.user;
                  if (user && user.authenticated && user.token) {
                      app.set("user", user);
                      app.set("validateuser",true);
                      if(user.properties && user.properties.length > 1){
                          /* setting the property list for the accordion
                           */
                          app.set("moreProperties", true);
                          that.transitionToRoute("selectproperty");
                          Ember.$.getJSON(App.HostUrl + "/properties").then(function(totalproperties){
                              if(totalproperties && totalproperties.properties){
                                  var allproperties = user.properties
                                  var statewiseProperty = [];
                                  var propertyObj = {};
                                  var stateObj = {};
                                  for(var i = 0; i < allproperties.length; i++){
                                      propObj = _.findWhere(totalproperties.properties, {"id": allproperties[i]});
                                      stateObj = _.findWhere(app.get("states"), {"abbr": propObj.state});
                                      if(!_.findWhere(statewiseProperty, {"name": stateObj.name})){
                                          var propertyObj = {
                                              "id": propObj.name,
                                              "name": propObj.name,
                                              "type": "property"
                                          };

                                          statewiseProperty.push({
                                              "id": stateObj.name,
                                              "children": [propertyObj],
                                              "name": stateObj.name,
                                              "type": "state"
                                          });
                                      } else {
                                          var stateResult = _.findWhere(statewiseProperty, {"name": stateObj.name});
                                          var propertyObj = {
                                              "id": propObj.name,
                                              "name": propObj.name,
                                              "type": "property"
                                          };
                                          stateResult.children.push(propertyObj);
                                      }
                                  }
                                  app.set("allproperties", statewiseProperty);
                                  $(".spinner").hide();
                              }
                          });
                      } else {
                          app.set("moreProperties", false);
                          app.send("propertyClick", user.properties[0]);
                          app.set("currentProperty", user.properties[0]);
                          that.transitionToRoute("/" + user.properties[0] + "home");
                          $(".spinner").hide();
                      }
                      $(".modal-backdrop").remove();
                      //get property list of logined user
                      $('.closemodal').click();
                      window.onbeforeunload = function(){
                          return "You are still working on the property.";
                      };
                      var d = that.get("destination");
                      if (d) {
                          d.retry();
                      } else {
                          // otherwise go to default users page
                          if(user.properties && user.properties.length > 1){
                              that.transitionToRoute("selectproperty");
                          } else{
                              that.transitionToRoute("/" + user.properties[0] + "/home");
                          }
                      }
                  }
              } catch (e) {
                  that.set("errorMessage", "An unknown error occurred, please try again. (" + e.message + ")");
              }
          },
          error: function(data) {
              $(".spinner").hide();
              that.set("errorMessage", data.responseText);
          }
      });
  } else {
      this.set("errorMessage", "The username and password are required.");
  }
}
```



## Template of Document Section
> Here, the user can View/Create/Delete documents

```
<div class="container">
    <div class="parent_section_tab resp-vtabs dasboard_tab" style="auto">
        <!--tab right pannel start-->
        <div class="resp-tabs-container hor_1" style="border-color: rgb(193, 193, 193);">
            
            <div class="resp-tab-content hor_1 resp-tab-content-active" aria-labelledby="hor_1_tab_item-2" style="display:block; border: none;">
                <div class="document btnpage demo">
                    <ul>
                        {{#each documentsList as |document|}}
                        <li>
                            <a href={{document.filePath}} target="_blank">
                                <div class="box">
                                    {{input type="checkbox" checked=document.selected classNames="document-checkbox"}}<figure><img height="64px" width="64px" src={{concat (is-filetype document.fileName true)}} alt="pdf_image" /></figure>
                                    <a href={{document.filePath}} target="_blank">{{document.fileName}}</a>
                                </div>
                            </a>
                        </li>
                        {{/each}}
                    </ul>
                    <div class="" style="padding-top: 30px">
                        {{file-uploader id="upload-a-document" content=this name="fileName" classNames="fileUploader" visible="preview" src="previewSrc"}}
                        <button class="btn btn-default" onclick="$('input[id=upload-a-document]').click();">Upload Document</button>
                        <button class="btn btn-default" {{action "deleteDocuments"}}>Delete Document</button>
                        {{#if applicationController.mobileuser}} {{file-uploader id="take-a-photo" content=this name="fileName" classNames="fileUploader" visible="preview" src="previewSrc"}}
                        <button class="btn btn-default" onclick="$('input[id=take-a-photo]').click();">Take Photo</button>
                        {{/if}}
                    </div>
                </div>
            </div>
        </div>
    </div>
    <!--<div id="addtohome-menu" {{action "addPhotoToHome"}}>Make this photo the Home section photo default</div>
    <div class="photo-overlay"></div>
    <div class="overlay-image">
        <img src="" width="100%" height="100%" style="max-height: 446px;" />
        <div class="close close-manual" {{action "closePhotoOverlay"}}>x</div>
        <div id="name-of-file" style="color: #000; padding: 4px; text-align: center; font-size: x-large"></div>
    </div>-->
</div>

```



## Template of Extrafeatures Section
> The user can update the features and upload some images with the features too.

```
<div class="parent_section_tab">
    <div id="svg_cont" style="position:relative; top:0; left:0; width:100%;height:100%;">
        <div id="erRect">
            <div style="margin:10px;">
                <img src="images/xls.png" style="width: 50px; margin: 0 10px; cursor: pointer;" title="Export view as excel" {{action "exportExcelFile" extrafeatures}} class="" id="excelShape">
                {{#if isDesigner}} {{#if isRoleForDE}}
                <button class="erPageSaveButton btn btn-primary" {{action "showInsertField" }}>Insert New Row</button>
                {{/if}}{{/if}}
                <button class="erPageSaveButton btn btn-success" {{action "updationData" }}>Save</button>
                <button class="erPageSaveButton btn btn-warning" {{action "saveOnCancel" }}>Cancel</button>
            </div>

            {{#if insertNewRow}}
                {{insert-row extrafeatures=extrafeatures store=store currentCompany = currentCompany currentProperty = currentProperty currentSection=currentSection currentPage = currentPage updateIsEditing = updateIsEditing extrafeaturesKeys=extrafeaturesKeys insertNewRow=insertNewRow errorMsg=errorMsg}}
            {{/if}}

            <table class="table table-striped table-bordered table-condensed">
                <thead>
                    <tr>
                        <th colspan="4" style="font-size:18px;font-weight:bold; text-align: center; line-height: 30px;">FEATURES</th>
                    </tr>
                </thead>
                <tbody>
                    {{#each extrafeatures as |item|}}
                        <tr style="border-bottom-style: double;">
                            <td id="erName" class="col-md-2 text-center vertical-align">{{item.name}}</td>
                            <td style="color: #FF6969;" class="col-md-4">
                                {{#if isDesigner}} {{#if isRoleForDE}} {{#inline-edit value=value extrafeatures=extrafeatures store=store currentCompany = currentCompany currentProperty = currentProperty currentSection=currentSection currentPage = currentPage updateIsEditing = updateIsEditing item=item}} {{textarea value=item.description classNames="form-control" }} {{/inline-edit}} {{else}}{{{item.htmlDescription }}} {{/if}}{{/if}}
                            </td>
                            <td class="col-md-4">
                                {{#each item.files as |image_url|}}
                                    <img src={{image_url}} class="thumb-image-small">
                                {{/each}}
                            </td>
                            <td class="col-md-2 text-center vertical-align">
                                <label class="btn btn-info btn-sm glyphicon glyphicon-folder-open" title="Browse images">
                                    {{file-uploader content=this id=item.name name="files" classNames="hide" visible="preview" src="previewSrc" multiple="multiple"}}
                                </label>
                                &#160;
                                <span class="btn btn-success btn-sm glyphicon glyphicon-plus" {{action "addFiles" item}} title="Add selected images"></span>
                            </td>
                        </tr>
                    {{/each}}
                </tbody>
            </table>
        </div>
    </div>
</div>
```



## Template of a section having canvas
> We are having many section eg. Plumbing, Space, Background etc where the user can draw shapes according to the building architecture.

```
<div id="content">
    <div id="svg_cont" style="position:relative; top:0; left:0; width:100%;height:100%;">
        <svg id="svg" class="svg-main" width="100%" height="100%" viewBox={{pageViewbox}} preserveAspectRatio="xMidYMid meet">
            <defs></defs>
            <g id="layerWrapper" class="svg-view">
                <!-- layer background component which makes background and foreground layer and show shapes and stencils according to the current company,property , section and page filter  -->
                {{layer-background currentCompany = currentCompany currentProperty = currentProperty currentSection = currentSection currentPage = currentPage newShapeObj = newShapeObj stencilObj=stencilObj newPageObj = newPageObj pageSettingObj = pageSettingObj pageWidth = pageWidth pageHeight = pageHeight pageData = pageData allassets = allassets pageSvgBackgrounds =pageSvgBackgrounds backgroundLayer = backgroundLayer foregroundShapes = foregroundShapes tagName = "svg" currentTool=currentTool pageBoundaryPath =pageBoundaryPath onAssetClick=onAssetClick parentThis=this fetchRefreshData = fetchRefreshData refreshBoolean = refreshBoolean classNames = currentSection isLegends=isLegends totalPagesCount=totalPagesCount isDesignRole=isDesignRole currentPropertyData=currentPropertyData showPropInfo=showPropInfo}}
            </g>
            {{#if showPropInfo}}
                {{prop-info id="propInfo" tagName="svg" currentPropertyData=currentPropertyData currentCompany =currentCompany companyLogo=companyLogo companyAddress=companyAddress pageViewbox=pageViewbox}}
            {{/if}}
        </svg>
    </div>
</div>
<div class="popup_wrapper">
    {{label-popup id="label-popup"popupshape=popupshape isDesignMode = isDesignMode namedColors=namedColors viewBool=viewBool alignText=alignText}}
    {{#if shapeProps.showProps}} {{#if showPopup}}{{shape-popup shapeProps = shapeProps infoBoxOptions = infoBoxOptions isDesignMode = isDesignMode isFullScreen = isFullScreen}}{{/if}} {{/if}}
    {{#pop-over columns=columns stencilid=stencilid isDesignMode=isDesignMode viewBool=viewBool isProtected=isProtected isReader=isReader stencilObj=stencilObj isDetails = isDetails isPhotos = isPhotos isUploads = isUploads isDesignRole=isDesignRole id="popup" colorPicker=colorPicker}}{{/pop-over}}
</div>
```



## Arc Tool

> A line can be converted to an arc with the help of this tool, a red circle appears on the line and if we have a closed shape, a red circle appears on each edge of the shape.
> Clicking and dragging the circle makes the arc of the line with the desired angle.

```
arcBehavior: function() {
    var that = this;
    var pathD; //store path D attribute value for using in scale behaviour like in drag and end
    var pointName; //Handler name which is currently active
    var pathObj; // svg path object we change here the new D attribute values
    var canvas = d3.select(".svg-foregroundShapes").node(); // canvas drawing area
    var objectId; //currently selected element 
    var selectedPointObject;
    var dx, dy;
    var sweepFlag = 0;
    var svgView = document.querySelector("#svg");
    var pt, g, transformg;
    var isDoubleClick = false;
    var clickCount = 0; // For click count
    var dragStartPoint = {};
    var pointCollection = [];
    
    return d3.behavior.drag()
        .on("dragstart", function() {
            clickCount++;
            // Double click event is not working so we use clickCount varible for checking double click.
            if (clickCount === 1) {
                isDoubleClick = false;
                setTimeout(function() {
                    clickCount = 0;
                }, 400);
            } else if (clickCount === 2) {
                clickCount = 0;
                // set true for double click
                isDoubleClick = true;
            }
            if (!isDoubleClick) {

                pt = svgView.createSVGPoint();
                canvas = d3.select(".svg-foregroundShapes").node();
                dx = 0;
                dy = 0;
                d3.event.sourceEvent.stopPropagation();
                d3.event.sourceEvent.preventDefault();
                dragStartPoint.x = d3.mouse(canvas)[0];
                dragStartPoint.y = d3.mouse(canvas)[1];
                // hide handles so outline can be seen in corners and midway
                d3.selectAll(".idp-selected-handle").style("visibility", "hidden");
                d3.selectAll("idp-selected-g").remove();
                // prevent premature de-selection
                d3.select("body").classed("idp-has-selection", true);
                objectId = d3.select(this.parentNode).attr('selectedObjectId');
                var childobid = d3.select("#" + objectId).select("path").attr("irrigchildlineid");
                var handle = d3.select('#' + objectId);
                // select required path d of stencil
                if (handle.attr && handle.attr('type') && handle.attr('type') == "stencil") {
                    pathD = handle.select('g').selectAll('path').filter(function(d, i) {
                        return this.id == "";
                    }).attr('d');
                } else {
                    pathD = handle.select('g').select('path').attr('d');
                }
                g = handle.select('g');
                transformg = Utils.SVG.parseTransformString(g.attr("transform"));
                that.set('beforeChange', pathD);
                // select required path d of stencil
                if (handle.attr && handle.attr('type') && handle.attr('type') == "stencil") {
                    pathObj = handle.select('g').selectAll('path').filter(function(d, i) {
                        return this.id == "";
                    });
                } else {
                    pathObj = handle.select('g').select('path');
                }
                var strokeWidth = pathObj.attr('stroke-width');
                pointName = d3.select(this.parentNode).attr('pointName');
                var r = d3.select(".idp-selected").select(".idp-selected");
                selectedPointObject = that.get("midPointArray").findBy("id", pointName);
                var parent2 = selectedPointObject["parent2"];
                var parent1 = selectedPointObject["parent1"];
                var rx, ry, rotationAngle, Lx, Ly;
                if (parent2[0].toUpperCase() == "L") {
                    if (parent1[0].toUpperCase() == "A") {
                        rotationAngle = Utils.SVG.findAngleOfTwoPoints({
                            "x": parent1[6],
                            "y": parent1[7]
                        }, {
                            "x": parent2[1],
                            "y": parent2[2]
                        });
                        var rAngle = rotationAngle;
                        if (rAngle < 0)
                            rAngle = 360 + rAngle;
                        if ((rAngle > 45 && rAngle <= 135) || (rAngle > 225 && rAngle <= 315)) {
                            //Its Vertical Line - Use Old calculation //Fix YRadii
                            rx = Math.sqrt(Math.pow((parent1[6] - parent2[1]), 2) + Math.pow((parent1[7] - parent2[2]), 2));
                            rx = parseFloat(rx / 2);
                            ry = 1;
                        } else {
                            //Its Horizontal Line - Fix the XRadii
                            rx = Math.abs(parent2[1] - parent1[6]) / 2 + (2 * strokeWidth);
                            ry = 1;
                        }

                        //Old Code - fixes always XRadii
                        rx = parent1[6] + parent2[1];
                        ry = rx;

                    } else if (parent1[0].toUpperCase() == "M" || parent1[0].toUpperCase() == "L") {
                        rotationAngle = Utils.SVG.findAngleOfTwoPoints({
                            "x": parent1[1],
                            "y": parent1[2]
                        }, {
                            "x": parent2[1],
                            "y": parent2[2]
                        });
                        var rAngle = rotationAngle;
                        if (rAngle < 0)
                            rAngle = 360 + rAngle;
                        if ((rAngle > 45 && rAngle <= 135) || (rAngle > 225 && rAngle <= 315)) {
                            //Its Vertical Line - Use Old calculation //Fix YRadii
                            rx = Math.sqrt(Math.pow((parent2[1] - parent1[1]), 2) + Math.pow((parent2[2] - parent1[2]), 2));
                            rx = parseFloat(rx / 2);
                            ry = 1;
                        } else {
                            //Its Horizontal Line - Fix the XRadii
                            rx = Math.abs(parent2[1] - parent1[1]) / 2 + (2 * strokeWidth);
                            ry = 1;
                        }
                        //Old Code - fixes always XRadii
                        rx = parent2[1] + parent1[1];
                        ry = rx;
                    }
                    Lx = parent2[1];
                    Ly = parent2[2];
                } else if (parent2[0] == "A") {
                    rx = parent2[1];
                    ry = parent2[2];
                    rotationAngle = parent2[3];
                    sweepFlag = parent2[5];
                    Lx = parent2[6];
                    Ly = parent2[7];
                }
                rotationAngle = 0;
                selectedPointObject["arc"] = "A" + rx + "," + ry + " " + rotationAngle + " 0," + sweepFlag + " " + Lx + "," + Ly;
                //Test : SET ANGLE to 0 -- //selectedPointObject["arc"] = "A" + rx + "," + ry + " " + "0" + " 0," + sweepFlag + " " + Lx + "," + Ly;
                //Changes to digest the different path styles (with spaces and with commas)
                if (pathD.indexOf(parent2.join(" ")) > -1) {
                    parent2 = parent2.join(" ");
                    pathD = pathD.replace(parent2, selectedPointObject.arc);
                } else {
                    parent2 = parent2.join(",");
                    parent2 = parent2.slice(0, 1) + parent2.slice(2, parent2.length);
                    pathD = pathD.replace(/ /g, ",");
                    pathD = pathD.replace(parent2, selectedPointObject.arc);
                }
                selectedPointObject["parent2"] = Snap.parsePathString(selectedPointObject["arc"])[0];
            }
        })
        .on("drag", function() {
            d3.event.sourceEvent.stopPropagation();
            d3.event.sourceEvent.preventDefault();
            if (!isDoubleClick) {

                dx += d3.event.dx;
                dy += d3.event.dy;
                var points = Snap.parsePathString(pathD);
                var arcChange = "";
                var length = points.length;
                var changedArc, index;
                changedArc = selectedPointObject["arc"];
                index = changedArc.indexOf(",");
                var arcPathArr = Snap.parsePathString(changedArc);
                var mymidpoint;
                var mid_point;
                var ellipseAngle = 0;

                var x1 = 0;
                var y1 = 0;
                var x2 = 0;
                var y2 = 0;

                if (selectedPointObject["parent1"][0].toUpperCase() == "L" || selectedPointObject["parent1"][0].toUpperCase() == "M") {
                    mid_point = { "x": transformg.translate.x + (transformg.scale.x * (parseFloat(selectedPointObject["parent1"][1]) + Number(selectedPointObject["parent2"][6]))) / 2, "y": transformg.translate.y + (transformg.scale.y * (parseFloat(selectedPointObject["parent1"][2]) + parseFloat(selectedPointObject["parent2"][7]))) / 2 }

                    ellipseAngle = Utils.SVG.findAngleOfTwoPoints({
                        "x": selectedPointObject["parent1"][1],
                        "y": selectedPointObject["parent1"][2]
                    }, {
                        "x": selectedPointObject["parent2"][6],
                        "y": selectedPointObject["parent2"][7]
                    });
                    
                    mymidpoint = { "x": (parseFloat(selectedPointObject["parent1"][1]) + Number(selectedPointObject["parent2"][6])) / 2, "y": (parseFloat(selectedPointObject["parent1"][2]) + parseFloat(selectedPointObject["parent2"][7])) / 2 };

                    x1 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent1"][1]);
                    y1 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent1"][2]);
                    x2 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent2"][6]);
                    y2 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent2"][7]);

                } else if (selectedPointObject["parent1"][0] == "A") {
                    mid_point = { "x": transformg.translate.x + (transformg.scale.x * (parseFloat(selectedPointObject["parent1"][6]) + Number(selectedPointObject["parent2"][6]))) / 2, "y": transformg.translate.y + (transformg.scale.y * (parseFloat(selectedPointObject["parent1"][7]) + parseFloat(selectedPointObject["parent2"][7]))) / 2 }

                    ellipseAngle = Utils.SVG.findAngleOfTwoPoints({
                        "x": selectedPointObject["parent1"][6],
                        "y": selectedPointObject["parent1"][7]
                    }, {
                        "x": selectedPointObject["parent2"][6],
                        "y": selectedPointObject["parent2"][7]
                    });
                    
                    mymidpoint = { "x": (parseFloat(selectedPointObject["parent1"][6]) + Number(selectedPointObject["parent2"][6])) / 2, "y": (parseFloat(selectedPointObject["parent1"][7]) + parseFloat(selectedPointObject["parent2"][7])) / 2 };

                    x1 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent1"][6]);
                    y1 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent1"][7]);
                    x2 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent2"][6]);
                    y2 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent2"][7]);
                } else {
                    mid_point = { "x": transformg.translate.x + (transformg.scale.x * (parseFloat(selectedPointObject["parent1"][1]) + Number(selectedPointObject["parent2"][1]))) / 2, "y": transformg.translate.y + (transformg.scale.y * (parseFloat(selectedPointObject["parent1"][2]) + parseFloat(selectedPointObject["parent2"][2]))) / 2 }
                    mymidpoint = { "x": (parseFloat(selectedPointObject["parent1"][1]) + Number(selectedPointObject["parent2"][1])) / 2, "y": (parseFloat(selectedPointObject["parent1"][2]) + parseFloat(selectedPointObject["parent2"][2])) / 2 };

                    x1 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent1"][1]);
                    y1 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent1"][2]);
                    x2 = transformg.translate.x + transformg.scale.x * parseFloat(selectedPointObject["parent2"][1]);
                    y2 = transformg.translate.y + transformg.scale.y * parseFloat(selectedPointObject["parent2"][2]);
                }
                mid_point = that.getUpdatedPoint(mymidpoint.x, mymidpoint.y, transformg, g);

                var rotateTransformR = transformg.rotate.r;
                var rotateTransformX = transformg.rotate.x;
                var rotateTransformY = transformg.rotate.y;

                //console.log('RotateTransform R= ' + rotateTransformR + ' X= ' + rotateTransformX + ' Y= ' + rotateTransformY);

                var mouseX = d3.mouse(canvas)[0];
                var mouseY = d3.mouse(canvas)[1];

                var theta = -rotateTransformR * (Math.PI / 180);
                var pointToRotate = { "x": mouseX, "y": mouseY };

                var arcAngle = arcPathArr[0][3];
                var angle = arcAngle;

                var PointX = mouseX - mid_point.x;
                var PointY = mouseY - mid_point.y;

                if (angle < 0) {
                    angle = 360 + angle;
                }

                if ((angle > 45 && angle <= 135) || (angle > 225 && angle <= 315)) {
                    //Item#1070 - ARCS and vertex tool - vertical line midpoint dragging faster than mouse.
                    PointX = mouseX - arcPathArr[0][6];
                    PointY = mouseY - arcPathArr[0][7];
                }

                var ValueOfB = Math.abs(Math.pow(PointY, 2) / (1 - (Math.pow(PointX, 2) / Math.pow(arcPathArr[0][1], 2))));

                var mouseAngle = Utils.SVG.findAngleOfTwoPoints({
                    "x": 0,
                    "y": 0
                }, {
                    "x": PointX,
                    "y": PointY
                });

                //Changes for Circular Arc
                var arcDepth = Math.sqrt(Math.pow(mouseX - mid_point.x, 2) + Math.pow(mouseY - mid_point.y, 2));
                var halfSpan = (Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2))) / 2;

                var newRx = (((arcDepth * arcDepth) + (halfSpan * halfSpan)) / (2 * arcDepth));
                arcPathArr[0][1] = newRx;
                arcPathArr[0][2] = newRx;

                if (arcDepth > halfSpan) {
                    arcPathArr[0][4] = 1;
                } else {
                    arcPathArr[0][4] = 0;
                }

                var pointCollection = [];

                var SP = [];
                if (selectedPointObject["parent1"][0] == "A") {
                    SP.push({ 'X': selectedPointObject["parent1"][6] }, { 'Y': selectedPointObject["parent1"][7] });
                    var ep1 = that.getUpdatedPoint(selectedPointObject["parent1"][6], selectedPointObject["parent1"][7], transformg, g);
                    pointCollection.push({ 'P1': [{ 'X': ep1.x }, { 'Y': ep1.y }] });
                } else {
                    SP.push({ 'X': selectedPointObject["parent1"][1] }, { 'Y': selectedPointObject["parent1"][2] });
                    var ep1 = that.getUpdatedPoint(selectedPointObject["parent1"][1], selectedPointObject["parent1"][2], transformg, g);
                    pointCollection.push({ 'P1': [{ 'X': ep1.x }, { 'Y': ep1.y }] });
                }

                pointCollection.push({ 'P2': [{ 'X': mouseX }, { 'Y': mouseY }] });

                if (selectedPointObject["parent2"][0] == "A") {
                    var ep3 = that.getUpdatedPoint(selectedPointObject["parent2"][6], selectedPointObject["parent2"][7], transformg, g);
                    pointCollection.push({ 'P3': [{ 'X': ep3.x }, { 'Y': ep3.y }] });
                } else {
                    var ep3 = that.getUpdatedPoint(selectedPointObject["parent2"][1], selectedPointObject["parent2"][2], transformg, g);
                    pointCollection.push({ 'P3': [{ 'X': ep3.x }, { 'Y': ep3.y }] });
                }

                var ep4 = that.getUpdatedPoint(SP[0].X, SP[1].Y, transformg, g);
                pointCollection.push({ 'P4': [{ 'X': ep4.x }, { 'Y': ep4.y }] });

                arcPathArr[0][5] = that.getFlag(pointCollection);

                var diff1 = Math.sqrt(parseFloat(Math.pow((parseFloat(mouseX.toFixed(2)) - selectedPointObject["mid"][1]), 2).toFixed(2)) + Math.pow((parseFloat((d3.mouse(canvas)[1]).toFixed(2)) - selectedPointObject["mid"][2]), 2)).toFixed(2);
                changedArc = arcPathArr[0][0] + arcPathArr[0][1] + "," + (parseFloat(arcPathArr[0][2])) + " " + arcPathArr[0][3] + " " + arcPathArr[0][4] + "," + arcPathArr[0][5] + " " + arcPathArr[0][6] + "," + arcPathArr[0][7];
                pathD = pathD.replace(selectedPointObject.arc, changedArc);
                selectedPointObject["arc"] = changedArc;
                selectedPointObject["parent2"] = Snap.parsePathString(changedArc)[0];
                pathObj.attr({
                    'd': pathD
                });

                // if on irrig section  wire path  is created using path tool 
                // and arc tool is apply on wire path then child path will also be change  
                var ele = d3.select("#" + objectId).select("g").select("path")[0];
                var childpathid = ele[0] && ele[0].attributes && ele[0].attributes["irrigchildlineid"] && ele[0].attributes["irrigchildlineid"]["value"];
                if (childpathid) {
                    var savedToObject = that.store.peekRecord(that.get("itemShape"), childpathid);
                    if (savedToObject) {
                        savedToObject.set("d", pathObj.attr('d'));
                    }
                }
            }
        })
        .on("dragend", function(evt) {
            d3.event.sourceEvent.stopPropagation();
            d3.event.sourceEvent.preventDefault();
            if (!isDoubleClick) {

                // if on irrig section  wire path  is created using path tool 
                // and arc tool is apply on wire path then child path will also be change
                // and after dragend child path store to the database
                var ele = d3.select("#" + objectId).select("g").select("path")[0];
                var childpathid = ele[0] && ele[0].attributes && ele[0].attributes["irrigchildlineid"] && ele[0].attributes["irrigchildlineid"]["value"];
                if (childpathid) {
                    var savedToObject = that.store.peekRecord(that.get("itemShape"), childpathid);
                    if (savedToObject) {
                        savedToObject.set("d", pathObj.attr('d'));
                        savedToObject.save().then(function(daa) {});
                    }
                }

                /*  MUST BE LAST
                    we do this because after the drag ends, if the mouse cursor is over background svg
                    it de-selects the shape(s) and it could be off just because of mouse movement
                    and the user can't click on background purposefully in under 200 ms after scaling a shape
                    so this should be safe to do and allow the selection to be maintained
                */
                var savedToObject = _.findWhere(that.get("allassets"), { "id": objectId });
                savedToObject.set("d", pathObj.attr('d'));
                // Checking if shape has some changes to set in undo array or not
                if (!savedToObject.get("hasDirtyAttributes")) {
                    that.set("addToUNDO", false);
                } else {
                    that.set("addToUNDO", true);
                }
                savedToObject.save().then(function(daa) {
                    that.set("addToUNDO", true);
                    var points = Snap.parsePathString(pathObj.attr("d"));
                    var startPoints = {
                        x: points[0][1],
                        y: points[0][2]
                    };
                    var len = points.length;
                    var midHandle = [];
                    var midPointArray = [];
                    for (var j = 0; j < len - 1; j++) {
                        if (points[j][0] == "A" || points[j + 1][0] == "A") {
                            if (points[j + 1][0] == "A" && points[j][0] == "A") {
                                var temp = points[j + 1][2];
                                points[j + 1][2] = Math.abs(points[j + 1][2])
                                var d = "M" + points[j][6] + "," + points[j][7] + points[j + 1].join(" ");
                                var totalLength = Snap.path.getTotalLength(d);
                                var midPoint = Snap.path.getPointAtLength(d, totalLength / 2)
                                ratio = (points[j + 1][1]) ? (points[j + 1][2] / (points[j + 1][1])) : 0;
                                points[j + 1][2] = temp;
                                x = midPoint.x;
                                y = midPoint.y;
                            } else if (points[j + 1][0] == "A") {
                                var temp = points[j + 1][2];
                                points[j + 1][2] = Math.abs(points[j + 1][2])
                                var d = "M" + points[j][1] + "," + points[j][2] + points[j + 1].join(" ");
                                var totalLength = Snap.path.getTotalLength(d);
                                ratio = (points[j + 1][1]) ? (points[j + 1][2] / (points[j + 1][1])) : 0;
                                midPoint = Snap.path.getPointAtLength(d, totalLength / 2);
                                points[j + 1][2] = temp;
                                x = midPoint.x;
                                y = midPoint.y;
                            } else {
                                if (points[j + 1]) {
                                    x = (points[j + 1][1] + points[j][6]) / 2;
                                    y = (points[j + 1][2] + points[j][7]) / 2
                                } else {
                                    x = (points[0][1] + points[j][6]) / 2;
                                    y = (points[0][2] + points[j][7]) / 2
                                }
                            }
                        } else {
                            x = (points[j + 1][1] + points[j][1]) / 2;
                            y = (points[j + 1][2] + points[j][2]) / 2;
                        }
                        midHandle = ["D", x, y];
                        midPointArray.push({
                            "parent1": points[j],
                            "parent2": points[j + 1] || points[0],
                            "mid": midHandle,
                            "id": "m" + j + "_1"
                        });
                    }
                    that.send("addHandleTemplate");
                    for (var i = 0; i < points.length; i++) {
                        that.send("addHandles", d3.select(".idp-selected-g").node(), points[i], startPoints, i, objectId, (points.length - 1) / 2);
                    }
                    for (var m = 0; m < midPointArray.length; m++) {
                        that.send("addHandles", d3.select(".idp-selected-g").node(), midPointArray[m]["mid"], startPoints, "m" + m, objectId)
                    }
                    that.set("midPointArray", midPointArray);
                }, function(err) {
                    that.set("addToUNDO", true);
                });
                Ember.run.later(function() {
                    d3.select('body').classed('idp-has-selection', false);
                }, 200);
                // to chage the value of applyVertexOrSelect to use it as observer.
                that.setObserver();
            } else {
                that.set("isFullScreen", false);
                var elem = d3.select("#" + objectId).select("g").select("path");
                selectedPointObject = that.get("midPointArray").findBy("id", pointName);
                var parent2 = selectedPointObject["parent2"];
                var parent1 = selectedPointObject["parent1"];
                // Check if clicked line is arc
                if (parent2.length && (parent2[0] == "A" || parent2[0] == "a")) {
                    var straightLine_D = "L" + parent2[6] + " " + parent2[7];
                    var d = pathObj.attr('d');
                    if (d.indexOf(parent2.join(" ")) > -1) {
                        parent2 = parent2.join(" ");
                        d = d.replace(parent2, straightLine_D);
                    } else {
                        parent2 = parent2.join(",");
                        parent2 = parent2.slice(0, 1) + parent2.slice(2, parent2.length);
                        d = d.replace(/ /g, ",");
                        d = d.replace(parent2, straightLine_D);
                    }
                    elem.attr({
                        'd': d
                    });
                    var savedToObject = _.findWhere(that.get("allassets"), { "id": objectId });
                    savedToObject.set("d", d);
                    savedToObject.save().then(function(daa) {
                        d3.selectAll(".idp-selected-handle").remove();
                        that.send("selectElement", d3.select('#' + objectId).node(), true);
                    });
                }
            }
        });
}
```
