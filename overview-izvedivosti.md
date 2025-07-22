# Izvedivost – Pregled

## 1. Konfiguracija Prikaza Poreza

**Izvedivost: VISOKA**

### Implementacija:

**WooCommerce → Settings → Tax:**
- Prikaži cijene u trgovini: Bez poreza
- Prikaži cijene tijekom košarice i naplate: S porezom
- Omogući opcije "Enable tax calculation" i "Display tax totals"

## 2. Mjesečni Izvještaji Prodaje po Zemljama

**Izvedivost: SREDNJA**

### Opcije:

#### WooCommerce Analytics + Proširenja
- WooCommerce Admin (osnovna) pruža osnovnu analitiku
- Potrebno je prilagođeno proširenje/izvještaj za mjesečni prikaz po zemljama

#### Vanjski Izvještaji:
- **Metorik** (napredna analitika) — $20-200/mjesec
- **Advanced WooCommerce Reporting plugin** — ~$49

#### Prilagođeno Rješenje za Izvještavanje:
- Prilagođeni widget za nadzornu ploču koristeći WP_Query i WC_Order
- Spremi podatke o porezu u meta podatke narudžbe za lakše upite

### Primjer Koda za Prilagođeni Izvještaj:

```php
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

    // Generiraj HTML/CSV izvještaj
}
```

## 3. PDF Računi s Informacijama o Porezu

**Izvedivost: VISOKA**

### Preporučena Rješenja:

#### WooCommerce PDF Invoices & Packing Slips 
- Besplatno/Premium €99
- Prilagodljivi predlošci, dodaj stopu poreza i informacije o zemlji

#### PDF Invoices for WooCommerce by WebToffee 
- $69
- Napredne značajke, bolje rukovanje informacijama o porezu

### Koraci Implementacije:
1. Instaliraj i aktiviraj preferirani plugin
2. Konfiguriraj predloške da uključuju:
   - Primijenjenu stopu poreza
   - Zemlju kupca
   - Razčlamba računanja poreza
3. Postavi automatsko generiranje nakon završetka narudžbe

## 4. Newsletter Popup s Nagradama

### Analiza Zahtjeva
- Popup za pretplatu na newsletter
- Besplatno PDF preuzimanje s pretplatom
- Jedinstveni jednokratni kod za popust (20%)
- Automatizirani e-mail dobrodošlice s PDF-om i kuponom

### Tehnička Izvedivost

#### Newsletter Popup Implementacija
**Izvedivost: VISOKA**

**Opcije:**

##### Integracija E-mail Marketinga:
- **Mailchimp for WooCommerce** (Besplatno)
- **ActiveCampaign for WooCommerce**

##### Dedicirani Popup Plugini:
- **Popup Maker** (Besplatno/Premium $87/godina)
- **OptinMonster** ($9–49/mjesec)
- **Hustle** (Besplatno/Premium)

**Preporučeno:** Integrirano rješenje poput Mailchimp + MC4WP

#### Generiranje Jedinstvenih Kupona
**Izvedivost: SREDNJA-VISOKA**

**Opcije Implementacije:**

##### Prilagođeno kod rješenje
Kačica u pretplatu, generiraj jedinstveni kupon

##### Plugin rješenja:
- **Advanced Coupons for WooCommerce** ($69-199)
- **WooCommerce Smart Coupons** ($99)

### Primjer Koda za Generiranje Kupona

```php
function generate_unique_subscriber_coupon($email) {
    $coupon_code = 'NEWS' . substr(md5(uniqid($email, true)), 0, 8);
    $coupon = new WC_Coupon();
    $coupon->set_code($coupon_code);
    $coupon->set_discount_type('percent');
    $coupon->set_amount(20); // 20% popust
    $coupon->set_individual_use(true);
    $coupon->set_usage_limit(1);
    $coupon->set_email_restrictions(array($email));
    $coupon->set_date_expires(strtotime('+30 days'));
    $coupon->save();
    return $coupon_code;
}

// Kačica za događaj slanja forme
add_action('mc4wp_form_subscribed', 'process_newsletter_subscription', 10, 3);
function process_newsletter_subscription($form, $args, $subscriber) {
    $email = $subscriber['EMAIL'];
    $coupon = generate_unique_subscriber_coupon($email);
    update_option('subscriber_coupon_' . md5($email), $coupon);
    send_welcome_email($email, $coupon);
}
```

#### Automatizirani E-mail Dobrodošlice s Prilozima
**Izvedivost: SREDNJA-VISOKA**

**Implementacija:**
- Koristi automatizaciju marketing platforme za e-mail/PDF/kupon
- Ili prilagođeni WP e-mail s wp_mail() i prilozima

**Razmotriti:**
- PDF spremanje s kontrolom pristupa
- Pouzdanost dostave e-maila (npr. SendGrid)
- Praćenje linkova

## 5. Lokalno Testno Okruženje

### Okruženje
- Local by Flywheel, XAMPP/MAMP, Docker

### Konfiguracija
- Najnoviji WP & WooCommerce
- Uzorni digitalni proizvodi

### Testiranje Poreznog Sustava
- Dodaj stope poreza po zemljama
- Testiraj s geolokacijom (proširenje preglednika ili kod):

```php
add_filter('woocommerce_geolocate_ip', function($ip_address) {
    return array(
        'country' => 'US', // Promijeni za testiranje
        'state' => 'NY',
    );
});
```

### Test Scenariji
- Stvori test narudžbe iz različitih lokacija
- Provjeri porez, generiranje PDF-a, dostavu e-maila, korištenje kupona

## 6. Potrebni Plugini i Resursi

### Osnovno: 
- WordPress, WooCommerce

### Porez: 
- WooCommerce PDF Invoices & Packing Slips, GeoIP plugin (opcionalno)

### Newsletter i Kupon: 
- Mailchimp for WooCommerce, MC4WP, ActiveCampaign, WP Mail SMTP, Advanced Coupons (opcionalno)

### Izvještavanje: 
- WooCommerce Admin, Metorik (opcionalno)

## 7. Procijenjeni Trud Implementacije

| Područje | Sati |
|----------|------|
| Osnovno WooCommerce Postavljanje | 2–4 |
| Konfiguracija Poreza | 2–3 |
| Geolokacija i Potvrda | 4–6 |
| Prilagodba PDF Računa | 2–3 |
| Newsletter Popup | 3–4 |
| Sustav Generiranja Kupona | 4–5 |
| Konfiguracija E-maila | 2–3 |
| Testiranje i Otklanjanje Grešaka | 6–8 |
| **Ukupno Procijenjeno** | **25–36** |

## 8. Zaključak

Oba zahtjeva su tehnički izvedljiva unutar WooCommerce ekosustava, iako složenost varira:

- **Upravljanje porezima:** Visoko izvedljivo s minimalnim prilagođenim kodom, koristeći WooCommerce osnove i plugine.
- **Newsletter popup s nagradama:** Umjereno složeno, ali ostvarivo s integracijom e-maila i prilagođenom logikom kupona.

Optimalna implementacija kombinira WooCommerce značajke, etablirane plugine i ciljani prilagođeni kod za efikasnost i održivost.