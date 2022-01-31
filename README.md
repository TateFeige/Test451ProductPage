<section class="product-detail-default">
    <loadingindicator ng-show="loadingIndicator" title="{{'Please wait while we fetch product information' | r | xlat}}" />
    <!-- BREADCRUMB!-->
    <productnav product="LineItem.Product" variant="LineItem.Variant" />
    <!-- PRODUCT IMAGE PANEL!-->
    <div class="col-md-4 col-collapse">
        <div class="panel panel-default">
            <div class="panel-body">
                <loadingindicator ng-show="loadingImage && LineItem.Variant.IsMpowerVariant" />
                <figure>
                    <img id="451_img_prod_lg" class="product-image-large img-responsive" ng-src="{{LineItem.Variant.PreviewUrl || LineItem.Variant.LargeImageUrl || LineItem.Product.LargeImageUrl}}" imageonload />
                </figure>
                <div class="empty" ng-hide="LineItem.Variant.PreviewUrl || LineItem.Variant.LargeImageUrl || LineItem.Product.LargeImageUrl">
                    <span class="fa empty"><i class="fa fa-camera"></i></span>
                </div>
            </div>
            <div class="panel-footer" ng-show="LineItem.Product.Type=='VariableText' && LineItem.Variant">
                <span class="btn-group">
                    <button ng-if="currentOrder.Status != 'AwaitingApproval' && !EditingLineItem" class="btn btn-default" redirect="/product/{{LineItem.Product.InteropID}}/{{LineItem.Variant.InteropID}}/edit">
                        {{'Edit' | r | xlat}}
                    </button>
	                <button ng-if="currentOrder.Status == 'AwaitingApproval' || EditingLineItem" class="btn btn-default" redirect="/product/{{LineItem.Product.InteropID}}/{{LineItem.Variant.InteropID}}/{{LineItemIndex}}/{{currentOrder.ID}}/edit">
                        {{'Edit' | r | xlat}}
                    </button>
                    <button class="btn btn-default" ng-if="currentOrder.Status != 'AwaitingApproval' && !EditingLineItem" ng-click="deleteVariant(LineItem.Variant, true)">
                        {{'Remove' | r | xlat}}
                    </button>
                    <button class="btn btn-default" ng-click="getPdf(LineItem.Variant.ProofUrl)">
                        {{('View' | r) + ' ' + ('PDF' | r) | xlat}}
                    </button>
                </span>
            </div>
        </div>
    </div>
    <div class="col-md-7 col-md-offset-1 col-collapse">
        <!-- PRODUCT INFO!-->
        <div class="row">
            <div class="col-xs-12">
                <h2 id="451qa_prod_name" ng-hide="LineItem.Variant.ExternalID">
                    {{LineItem.Product.Name}}
                </h2>
                <h2 ng-show="LineItem.Variant.ExternalID">
                    {{LineItem.Variant.ExternalID}}
                </h2>
                <h2 class="text-success" ng-show="LineItem.PriceSchedule.PriceBreaks.length == 1" ng-if="!(user.Permissions.contains('HidePricing'))">
                    {{LineItem.PriceSchedule.PriceBreaks[0].Price | culturecurrency}}
                </h2>
            </div>
        </div>
        <!-- PRODUCT DESCRIPTION PANEL! --->
        <div>
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title panel-primary">
                        {{'Details' | r | xlat}}
                    </h3>
                </div>
                <div class="panel-body" id="451_area_prod_desc">
                    <h4 ng-show="LineItem.Product.ExternalID" class="text-info">
                        {{'ID' | r | xlat}}: {{LineItem.Product.ExternalID}}
                    </h4>
                    <p id="451qa_prod_desc" ng-bind-html="trustedDescription(LineItem.Product)" />
                    <p ng-show="LineItem.Variant.Description" ng-bind-html="trustedDescription(LineItem.Variant)" />
                </div>
                <!-- PRICE SCHEDULE PANEL! --->
                <priceScheduleTable ng-if="!(user.Permissions.contains('HidePricing')) && LineItem.PriceSchedule.PriceBreaks.length > 1" ps='LineItem.PriceSchedule' p='LineItem.Product'/>
            </div>
        </div>
        <!-- STATIC SPECS PANEL! -->
        <staticspecstable ng-show="StaticSpecGroups && user.Permissions.contains('ViewNonCustomizableSpecs')" specgroups="StaticSpecGroups" length="LineItem.Product.StaticSpecLength" />
        <!-- ORDER PANEL! -->
        <form name="addToOrderForm" novalidate="">
            <div class="panel panel-default">
                <div class="panel-heading" ng-if="allowAddToOrder">
                    <h3 class="panel-title">
                        {{'Artwork & Quantities' | r | xlat}}
                    </h3>
                </div>
                <div class="panel-body">
                    <div class="alert alert-danger" ng-show="(!allowAddToOrder && !canCreateVariant && (!allowAddFromVariantList && (LineItem.Product.Variants.length == 0 || LineItem.Variant)))">
                        <p>
                            <i class="fa fa-ban" />
                            {{'Your current order does not allow this product to be added to your cart' | r | xlat}}
                        </p>
                    </div>
                    <!-- VARIANT LIST! -->
                    <div ng-if="!LineItem.Variant">
                        <div class="row">
                            <div class="panel-title col-xs-9" ng-show="(allowAddToOrder && LineItem.Product.Variants.length > 0) || canCreateVariant">
                                <h4 class="text-info">
                                    {{('Saved' | r) + ' ' + ('Artwork' | r) | xlat}}
                                </h4>
                            </div>
                            <div class="col-xs-3" ng-show="canCreateVariant">
                                <button type="button" ng-show="!LineItem.Variant" class="btn btn-danger pull-right" redirect="/product/{{LineItem.Product.InteropID}}/new/edit" tabindex="-1">
                                    {{'Create New' | r | xlat}}
                                </button>
                            </div>
                        </div>
                        <div class="view-form-icon" ng-if="LineItem.Product.VariantCount > settings.pageSize || searchTerm">
                            <div>
                                <div class="input-group">
                                    <input ng-model="searchTerm" type="text" class="form-control" placeholder="{{'Search' | r | xlat}}" />
                                    <i class="fa fa-search"></i>
                                    <div class="input-group-btn">
                                        <button type="button" class="btn btn-default" ng-click="searchVariants(searchTerm)">{{'Search' | r | xlat}}</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="product-variants">
                            <ul id="451qa_list_variants">
                                <loadingindicator ng-show="searchIndicator" />
                                <li ng-repeat="v in LineItem.Product.Variants | paginate:(settings.currentPage-1) * settings.pageSize | limitTo:settings.pageSize">
                                    <div class="row">
                                        <a ng-class="{'col-sm-6': allowAddFromVariantList, 'col-sm-11': !allowAddFromVariantList }" href=#/product/{{LineItem.Product.InteropID}}/{{v.InteropID}}>
                                            <h5>
                                                <strong>{{v.ExternalID}}</strong>
                                            </h5>
                                            <p ng-show="v.Description" ng-bind-html="trustedDescription(v)" />
                                            <p ng-show="LineItem.Product.DisplayInventory && LineItem.Product.IsVariantLevelInventory">
                                                {{'Quantity Available' | r | xlat}}: {{v.QuantityAvailable}}
                                            </p>
                                        </a>
                                        <div class="col-sm-2" ng-show="allowAddFromVariantList && !(user.Permissions.contains('HidePricing'))">
                                            <div class="quantity-total-variants text-center" ng-show="variantLineItems[v.InteropID].LineTotal > 0">
                                                <small><span class="text-info">{{'Total' | r | xlat}}</span><br /><span class=" text-success">{{variantLineItems[v.InteropID].LineTotal | culturecurrency}}</span></small>
                                            </div>
                                        </div>
                                        <div class="col-sm-3 view-form-icon" ng-show="allowAddFromVariantList">
                                            <quantityfield required="false" calculated="calcVariantLineItems" lineitem="variantLineItems[v.InteropID]" class="quantity" />
                                        </div>
                                        <div class="col-sm-1" ng-show="v.IsMpowerVariant">
                                            <button type="button" ng-click="deleteVariant(v, false)" class="btn btn-default btn-block">
                                                <i class="fa fa-trash-o text-warning"></i> <span class="visible-xs text-warning">{{'Delete' | r | xlat}}</span>
                                            </button>
                                        </div>
                                    </div>
                                </li>
                            </ul>
                        </div>
                        <div ng-show="LineItem.Product.VariantCount > settings.pageSize">
                            <pagination ng-change="paged()" ng-model="settings.currentPage" page="settings.currentPage" max-size="8" rotate="false" boundary-links="true" total-items="LineItem.Product.VariantCount"
                                        items-per-page="settings.pageSize" direction-links="true" previous-text="{{'Previous' | xlat}}" next-text="{{'Next' | xlat}}" first-text="{{'First' | xlat}}" last-text="{{'Last' | xlat}}"></pagination>
                        </div>
                    </div>
                    <ordertypeselector ng-show="(canAddToStandardOrder && canAddToReplenishmentOrder && !currentOrder.ID)"></ordertypeselector>
                    <addtoorderspecs ng-show="allowAddToOrder"></addtoorderspecs>
                    <div class="form-group" ng-show="allowAddToOrder">
                        <p ng-show="LineItem.Product.DisplayInventory && (LineItem.Variant || LineItem.Product.Variants.length == 0)">
                            <span class="text-info">{{'Quantity Available' | r | xlat}}</span> {{inventoryDisplay(LineItem.Product, LineItem.Variant)}}
                        </p>
                        <p ng-show="LineItem.PriceSchedule.MinQuantity > 1 && !LineItem.PriceSchedule.RestrictedQuantity">
                            <span class="text-info">{{'Minimum Quantity' | r | xlat}}</span> {{LineItem.PriceSchedule.MinQuantity}}
                        </p>
                        <p ng-show="LineItem.PriceSchedule.MaxQuantity && !LineItem.PriceSchedule.RestrictedQuantity">
                            <span class="text-info">{{'Maximum Quantity' | r | xlat}}</span> {{LineItem.PriceSchedule.MaxQuantity}}
                        </p>
                        <div class="view-form-icon" ng-if="!allowAddFromVariantList">
                            <div>
                                <label ng-class="{required: !allowAddFromVariantList}" ng-hide="allowAddFromVariantList">{{('Order' | r) + ' ' + ('Quantity' | r) | xlat}}</label>
                                <quantityfield required="true" lineitem="LineItem" class="quantity" />
                            </div>
                        </div>
                        <button class="btn btn-success btn-block btn-lg" type="button" id="451_btn_orderadd" ng-click="addToOrder()">
                            <loadingindicator ng-show="addToOrderIndicator" />
                            <i ng-show="lineItemErrors.length > 0" class="fa fa-warning"></i>
                            {{addToOrderText | r | xlat}}
                            <span id="451qa_lineitem_total" class="badge" ng-if="!(user.Permissions.contains('HidePricing')) && (LineItem.LineTotal || variantLineItemsOrderTotal) > 0">
                                {{(LineItem.LineTotal || variantLineItemsOrderTotal) | culturecurrency}}
                            </span>
                        </button>
                    </div>
                </div>
            </div>
        </form>
        <!-- RELATED PRODUCTS! -->
        <relatedproducts ng-if="LineItem.Product.RelatedProductsGroupID" relatedgroupid="LineItem.Product.RelatedProductsGroupID" />
    </div>
    <!-- ERROR MESSAGING! -->
    <div class="navbar-fixed-bottom">
        <div class="view-error-message" ng-show="showAddToCartErrors && lineItemErrors.length > 0">
            <ul class="alert-warning">
                <li class="text-center">
                    <span class="badge">
                        <i class="fa fa-exclamation-circle fa-inverse"></i>
                        {{lineItemErrors.length}}
                    </span>
                </li>
                <li class="text-center">
                    <ul ng-class="{'one': errorSection == 'open', 'two': errorSection == '' }">
                        <li class="alert-warning" ng-repeat="e in lineItemErrors">{{e | xlat}}</li>
                        <li class="alert-warning" ng-show="variantLineItems[v.InteropID].qtyError">{{variantLineItems[v.InteropID].qtyError}}</li>
                    </ul>
                </li>
                <li class="text-center">
                    <a ng-show="lineItemErrors.length > 1">
                        <i class="fa fa-caret-down" ng-click="errorSection = 'open'" ng-hide="errorSection == 'open'"></i>
                        <i class="fa fa-caret-up" ng-click="errorSection = ''" ng-show="errorSection == 'open'"></i>
                    </a>
                </li>
            </ul>
        </div>
    </div>
</section>
