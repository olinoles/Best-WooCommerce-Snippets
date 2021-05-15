# Best WooCommerce Snippets
A collection of WooCommerce PHP snippets written by me for adding more functionality to your online store.
All of these snippets are designed for WooCommerce, and tested up to version 4.8.0.  
They can be used by adding them to your functions.php file. I highly recommend using a [child theme](https://developer.wordpress.org/themes/advanced-topics/child-themes/) if this is your first time adding custom code!

- Snippets  
  - [List all brands alphabetically with links](#list-all-brands-alphabetically-with-links)  
  - [Add product page sections for each attribute](#add-product-page-sections-for-each-attribute)  
  - [Change default WooCommerce Placeholder image](#change-default-woocommerce-placeholder-image)  
  - [Add a list of Subcategories to any Category page](#add-a-list-of-subcategories-to-any-category-page)  
  - [Add a related products section to the bottom of product pages](#add-a-related-products-section-to-the-bottom-of-product-pages)
  - [Add a products in this Category section to the bottom of the product pages](#add-a-products-in-this-category-section-to-the-bottom-of-the-product-pages)
  - [Display the brand on the product page below the title](#display-the-brand-on-the-product-page-below-the-title)

## List all brands alphabetically with links
You can change instances of ```pa_brand``` to any other attribute name. You must use the *pa_* prefix before the attribute slug.
Some basic styling is included, but you can customise the look with CSS.  
You can call this shortcode using `[allbrands]`

```
function brands_shortcode() {
	$terms = get_terms("pa_brand");
		usort($terms, function($a, $b) {
			return strtoupper($a->name) > strtoupper($b->name) ? 1 : -1;
		});
	$term_first_letter = '';
	$triggered_new_cont = false;
	
	echo "<div class='brands-flex-cont'>";
	foreach ( $terms as $term ) {
		if($term_first_letter != $term->name[0]){
			$term_first_letter = strtoupper($term->name[0]);
			echo "</div>";
			echo "<div class='brands-first-letter-title' style='font-weight:600'>" . strtoupper($term_first_letter) . "</div>";
			echo "<div class='brands-flex-cont'>";
		}
		echo "<div class='cont-brands-link'><a class='brands-link' style='color:#148fcc;' href='" . get_term_link($term->slug,'pa_brand') . "'>" . $term->name . "</a></div>";
	}
	echo "</div>";
}

add_shortcode('allbrands', 'brands_shortcode');
```

## Add product page sections for each attribute
This piece of code allows you to create templates in Elementor and insert them on specific branded product pages. The example in the code inserts two different templates, depending on which brand the product is. The `$product_brand` must match the slug of the attribute you're using. Depending on where you want this on the product page, you can change the `10` at in the final parameter of the filter.

```
function brand_promo() { 
    global $product; 

    $product_brand = $product->get_attribute( 'brand' );
	
	$brand_terms = get_the_terms($post, 'pa_brand');
	$brand_string = ''; // Reset string
	foreach ($brand_terms as $term) :
    $brand_string = $term->slug;
	endforeach;

    // Display custom field under the title
    if($product_brand == "Samsung") {
    	echo do_shortcode('[elementor-template id="20669"]');
	}
	if($product_brand == "LG") {
    	echo do_shortcode('[elementor-template id="23172"]');
	}
	
}
add_filter( 'woocommerce_after_single_product_summary', 'brand_promo', 10 );
```

## Change default WooCommerce Placeholder image
Self explanatory. Change the placeholder image used on products which haven't had a picture set. Upload an image to your Wordpress uploads named "image-placeholder.jpg" to take effect.
```
add_filter('woocommerce_placeholder_img_src', 'custom_woocommerce_placeholder_img_src');

function custom_woocommerce_placeholder_img_src( $src ) {
	$upload_dir = wp_upload_dir();
	$uploads = untrailingslashit( $upload_dir['baseurl'] );
	// replace with path to your image
	$src = $uploads . '/image-placeholder.jpg';
	 
	return $src;
}
```

## Add a list of Subcategories to any Category page
You can call this shortcode on any category page using `[display_subcategories_list]`. It will display a list of subcategories of the current selected category. Great to put inside a sidebar or at the top of a category page. 

```
function displaySubcategoriesList() {
	if(is_search()){
		return;
	}
	$parentid = get_queried_object_id();
	$args = array(
    'parent' => $parentid
    );
    $terms = get_terms( 'product_cat', $args );
    
    if ( $terms ) {
		echo '<div class="subcats-list-con">';
			echo '<h5 class="subcat-title" style="margin-bottom:10px;">Subcategories</h5>';
			echo '<ul class="subcat-list">';
				foreach ( $terms as $term ) {
					echo '<li class="subcat">';
						echo '<a style="font-size:14px;" href="' .  esc_url( get_term_link( $term ) ) . '" class="' . $term->slug . '">';
							echo $term->name;
							echo '<span style="font-size:12px;color:#999;margin-left:4px;"> (' . $term->count . ')</span>';
							echo '</a>';                                                   
					echo '</li>';
			}
			echo '</ul>';
		echo '</div>';
		echo '<div style="border-top:1px dotted rgb(209,209,209);margin-top:10px;"></div>';
    }
}

add_shortcode('display_subcategories_list', 'displaySubcategoriesList');
```

## Add Pre-order text to any product page if the product is out of stock instead of backorder
This code adds more flexibility into which messages are shown on the product page, depending on how new the product is, and what the backorder status is.  
For example, if the product is new and out of stock the message "Pre-order now" will appear. Otherwise, a general "Out of stock" text. You can edit `$newness_days` to change your definition of new.
```
add_filter( 'woocommerce_get_availability', 'wcs_custom_get_availability', 1, 2);
function wcs_custom_get_availability( $availability, $_product ) {
	
	$newness_days = 30;
	$created = strtotime( $_product->get_date_created() );
	
    // Change In Stock Text
    if ( $_product->is_in_stock() ) {
		if ( $_product->get_stock_quantity() > 0 && $_product->get_stock_quantity() <= 3 &&  $_product->get_stock_quantity()) {
			$availability['availability'] = __('Hurry! Only ' . $_product->get_stock_quantity() . ' left in stock', 'woocommerce');
		} else if ( $_product->get_stock_quantity() <= 0 && $_product->is_on_backorder()) {
				$availability['availability'] = __('Pre-order now available.', 'woocommerce');
		} else {
			$availability['availability'] = __('In Stock at XXXXX', 'woocommerce');
		}
    } else if ( ( time() - ( 60 * 60 * 24 * $newness_days ) ) < $created ) {
		$availability['availability'] = __('Stock coming soon!', 'woocommerce');
	}
	else {
		$availability['availability'] = __('Out of stock', 'woocommerce');
		}
    return $availability;
}
```

## Add a 'New' Badge to any products in the carousel that are new
Adds a New badge to products using the same formatting as the default sale badge. You can customise in CSS using `.itsnew`.   
`$newness_days` can be edited to change your definition of new.

```
add_action( 'woocommerce_before_shop_loop_item_title', 'new_badge_shop_page', 10 );
          
function new_badge_shop_page() {
   global $product;
   $newness_days = 30;
   $created = strtotime( $product->get_date_created() );
   if ( ( time() - ( 60 * 60 * 24 * $newness_days ) ) < $created ) {
      echo '<span style="font-size:14px;line-height:14px;font-weight:600;background-color:#59A22F;border-radius:3px;color:#fff;z-index:100;position:absolute;top:0;left:0;margin:6px;padding:4px;box-shadow:0 1px 3px hsla(0, 0%, 0%, 0.2);" class="itsnew">New!</span>';
   }
}
```
## Add a related products section to the bottom of product pages
This script adds a row of 6 related products to the bottom of all product pages. Related products are found using associated tags. For example, if a product uses the tag *jerseys*, then other products with the tag *jerseys* will be shown at the bottom of the product pages. I highly encourage you to use several tags on each of your products! The tag is also clickable in the section, which takes you to that collection. Great snippet for increasing related product sales.
```
function add_tag_related_products_section() {
	global $product; 

  $terms = get_the_terms( $post->ID, 'product_tag' );
	$tag_slugs = array();
	$tag_names = array();
	if ( ! empty( $terms ) && ! is_wp_error( $terms ) ){
		foreach ( $terms as $term ) {
			$tag_slugs[] = $term->slug;
			$tag_names[] = $term->name;
		}
	}
	for ($i = 0; $i < count($tag_slugs); $i++) {
		echo '<div class="related products brand-recommended-products">';
		echo '<h2 style="margin-top:50px;">Other products in the collection ';
		echo '<a style="color:#13aff0;" class="highlighted-link" href="' . get_tag_link($terms[$i]) . '">' . $tag_names[$i] . '</a></h2>';
		echo do_shortcode('[products columns="6" orderby="rand" rows="1" tag="' .  $tag_slugs[$i]  . '" ]');
		echo '</div>';
	}
}

add_action( 'woocommerce_after_single_product', 'add_tag_related_products_section' );
```

## Add a products in this Category section to the bottom of the product pages
This snippet is similar to the one above, except it adds related products from that category instead. Combine with above snippet for great results.
```
function add_category_related_products_section() {
	global $product; 

  $terms = get_the_terms( $post->ID, 'product_cat' );
	$category_slugs = array();
	$category_names = array();
	if ( ! empty( $terms ) && ! is_wp_error( $terms ) ){
		foreach ( $terms as $term ) {
			$category_slugs[] = $term->slug;
			$category_names[] = $term->name;
		}
	}
	for ($i = 0; $i < count($category_slugs); $i++) {
		echo '<div class="related products brand-recommended-products">';
		echo '<h2 style="margin-top:50px;">Other ';
		echo '<a style="color:#13aff0;" class="highlighted-link" href="' . get_category_link($terms[$i]) . '">' . $category_names[$i] . '</a> you might like</h2>';
		echo do_shortcode('[products columns="6" orderby="rand" rows="1" category="' .  $category_slugs[$i]  . '" ]');
		echo '</div>';
	}
}

add_action( 'woocommerce_after_single_product', 'add_category_related_products_section' );
```

## Display the brand on the product page below the title
This snippet displays any attribute (I'm using brand name) below the title with a clickable link to the collection
```
function custom_action_after_single_product_title() { 
  global $product; 

  $product_brand = $product->get_attribute( 'brand' );
	$brand_terms = get_the_terms($post, 'pa_brand');
	$brand_string = ''; // Reset string
	foreach ($brand_terms as $term) :
    $brand_string = $term->slug;
	endforeach;

    // Display custom field under the title
    if($product_brand) {
    	echo '<div style="margin-bottom:20px;" class="product-brand-div">' . 'by ';
		echo '<a class="product-brand-a" href="' . get_term_link($brand_string,'pa_brand') . '">' . $product_brand . '</a></div>';
	}
}
add_filter( 'woocommerce_single_product_summary', 'custom_action_after_single_product_title', 5 );
```
