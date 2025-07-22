Izvedivost – Pregled
1. Tax Display Configuration
Feasibility: HIGH

Implementation:

WooCommerce → Settings → Tax:
Display prices in the shop: Excluding tax
Display prices during cart and checkout: Including tax
Enable "Enable tax calculation" and "Display tax totals" options
2. Monthly Sales Reports by Country
Feasibility: MEDIUM

Options:
WooCommerce Analytics + Extensions
WooCommerce Admin (core) provides basic analytics
Custom extension/report needed for country-specific monthly breakdown
Third-party Reporting:
Metorik (advanced analytics) — $20-200/month
Advanced WooCommerce Reporting plugin — ~$49
Custom Reporting Solution:
Custom dashboard widget using WP_Query and WC_Order
Store tax data in order meta for easier queries
Example Custom Report Code:
PHP
function country_sales_report() {
    $year = isset($_GET['year']) ? intval($_GET['year']) : date('Y');
    $months = range(1, 12);
    $countries = array();

    foreach ($months as $month) {
        $args = array(
            'post_type' => 'shop_order',
            'post_status' => 'wc-completed',
            'date_query' => array(
                array(
                    'year' => $year,
                    'month' => $month,
                ),
            ),
            'posts_per_page' => -1,
        );

        $orders = new WP_Query($args);

        if ($orders->have_posts()) {
            while ($orders->have_posts()) {
                $orders->the_post();
                $order = wc_get_order(get_the_ID());
                $country = $order->get_billing_country();

                if (!isset($countries[$country])) {
                    $countries[$country] = array_fill(0, 12, 0);
                }

                $countries[$country][$month-1] += $order->get_total();
            }
        }
        wp_reset_postdata();
    }

    // Output report HTML/CSV
}
3. PDF Invoices with Tax Information
Feasibility: HIGH

Recommended Solutions:
WooCommerce PDF Invoices & Packing Slips (Free/Premium €99)
Customizable templates, add tax rate & country info
PDF Invoices for WooCommerce by WebToffee ($69)
Advanced features, better tax info handling
Implementation Steps:
Install and activate preferred plugin
Configure templates to include:
Tax rate applied
Customer country
Tax calculation breakdown
Set automatic generation on order completion
4. Newsletter Popup with Rewards
Requirement Analysis
Popup for newsletter subscription
Free PDF download with subscription
Unique one-time discount code (20%)
Automated welcome email with PDF and coupon
Technical Feasibility
Newsletter Popup Implementation
Feasibility: HIGH

Options:

Email Marketing Integration:
Mailchimp for WooCommerce (Free)
ActiveCampaign for WooCommerce
Dedicated Popup Plugins:
Popup Maker (Free/Premium $87/year)
OptinMonster ($9–49/month)
Hustle (Free/Premium)
Recommended: Integrated solution like Mailchimp + MC4WP

Unique Coupon Generation
Feasibility: MEDIUM-HIGH

Implementation Options:

Custom code solution (hook into subscription, generate unique coupon)
Plugin solutions:
Advanced Coupons for WooCommerce ($69-199)
WooCommerce Smart Coupons ($99)
Example Coupon Generation Code
PHP
function generate_unique_subscriber_coupon($email) {
    $coupon_code = 'NEWS' . substr(md5(uniqid($email, true)), 0, 8);
    $coupon = new WC_Coupon();
    $coupon->set_code($coupon_code);
    $coupon->set_discount_type('percent');
    $coupon->set_amount(20); // 20% discount
    $coupon->set_individual_use(true);
    $coupon->set_usage_limit(1);
    $coupon->set_email_restrictions(array($email));
    $coupon->set_date_expires(strtotime('+30 days'));
    $coupon->save();
    return $coupon_code;
}

// Hook to form submission event
add_action('mc4wp_form_subscribed', 'process_newsletter_subscription', 10, 3);
function process_newsletter_subscription($form, $args, $subscriber) {
    $email = $subscriber['EMAIL'];
    $coupon = generate_unique_subscriber_coupon($email);
    update_option('subscriber_coupon_' . md5($email), $coupon);
    send_welcome_email($email, $coupon);
}
Automated Welcome Email with Attachments
Feasibility: MEDIUM-HIGH

Implementation:

Use marketing platform automation for email/PDF/coupon
Or custom WP email with wp_mail() and attachments
Considerations:

PDF storage with access control
Email deliverability (e.g. SendGrid)
Link tracking
5. Local Testing Setup
Environment
Local by Flywheel, XAMPP/MAMP, Docker
Configuration
Latest WP & WooCommerce
Sample digital products
Testing Tax System
Add country tax rates
Test with geolocation (browser extension or code):
PHP
add_filter('woocommerce_geolocate_ip', function($ip_address) {
    return array(
        'country' => 'US', // Change for testing
        'state' => 'NY',
    );
});
Test Scenarios
Create test orders from different locations
Verify tax, PDF generation, email delivery, coupon usage
6. Required Plugins & Resources
Core: WordPress, WooCommerce
Tax: WooCommerce PDF Invoices & Packing Slips, GeoIP plugin (optional)
Newsletter & Coupon: Mailchimp for WooCommerce, MC4WP, ActiveCampaign, WP Mail SMTP, Advanced Coupons (optional)
Reporting: WooCommerce Admin, Metorik (optional)
7. Estimated Implementation Effort
Area	Hours
Base WooCommerce Setup	2–4
Tax Configuration	2–3
Geolocation & Confirmation	4–6
PDF Invoice Customization	2–3
Newsletter Popup	3–4
Coupon Generation System	4–5
Email Configuration	2–3
Testing & Debugging	6–8
Total Estimated	25–36
8. Conclusion
Both requirements are technically feasible within the WooCommerce ecosystem, though complexity varies:

Tax management: Highly feasible with minimal custom code, leveraging WooCommerce core and plugins.
Newsletter popup with rewards: Moderately complex but achievable with email integration and custom coupon logic.
Optimal implementation combines WooCommerce features, established plugins, and targeted custom code for efficiency and maintainability.
