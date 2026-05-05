# SEO Technical Troubleshooting: Website Not Indexing

## Problem Statement

A new Worknoon website has been launched but is not appearing in Google search results even after submitting the XML sitemap to Google Search Console. This diagnostic guide provides systematic troubleshooting steps to identify and resolve indexing issues.

---

## Part 1: Quick Diagnosis (Immediate Actions)

### Step 1: Verify Basic Indexation Status

```bash
# Check if site is indexed at all
# In Google Search: site:worknoon.com

# Expected Result:
# Google should return some pages (homepage at minimum)

# If NO results appear:
# → Your site is NOT indexed (proceed to full diagnosis)

# If results appear:
# → Site IS partially indexed (check for coverage issues)
```

**Manual Check Method:**
1. Open Google Search
2. Type: `site:worknoon.com`
3. Review number of results shown
4. Note which pages appear/don't appear

**Search Console Check:**
- Go to Google Search Console
- Navigate to "Coverage" report
- Check three categories:
  - **Error** (pages that failed indexation)
  - **Valid with warnings** (pages indexed but with issues)
  - **Valid** (successfully indexed pages)

### Step 2: Check Recent Indexation in Search Console

**Location**: Google Search Console → Indexation → Pages

**What to Look For:**
```
Status Categories:

1. CRAWLED - NOT INDEXED
   → Page was found but not added to index
   → Likely causes:
      - Duplicate content
      - Thin content (< 300 words)
      - No-index directive
      - Canonicalization issues
      - Low content quality

2. EXCLUDED
   → Page deliberately excluded from index
   → Check reasons:
      - Robots.txt blocking
      - No-index tag/meta directive
      - Nofollow on crawl links
      - Duplicate of another page
      - Blocked by robots.txt

3. ERROR
   → Google couldn't crawl or index page
   → Types:
      - Server error (5xx)
      - Access denied (401, 403)
      - Not found (404)
      - Redirect error
```

### Step 3: Test with URL Inspection Tool

**In Google Search Console:**
1. Click "URL Inspection" (top search bar)
2. Enter homepage: `https://worknoon.com`
3. Click "Request Indexing"
4. Note any issues reported

**Expected Status: "URL is on Google"**

**Common Issues in Report:**
```
Issue: "Not on Google"
→ Page hasn't been indexed yet
→ May need manual request

Issue: "Crawled - Not Indexed"
→ Technical problem preventing indexation
→ See diagnostics below

Issue: "Duplicate without canonical"
→ Multiple versions of same page
→ Need canonical URL implementation
```

---

## Part 2: Crawlability Tests

### Test 2.1: Robots.txt Audit

**What is robots.txt?**
- File that tells Google which pages to crawl
- Located at: `https://worknoon.com/robots.txt`
- Critical for crawlability control

**Check Current robots.txt:**
```bash
# Method 1: Browser
# Visit https://worknoon.com/robots.txt
# Check what's displayed

# Method 2: WordPress Command
wp eval '$robots = wp_remote_get("https://worknoon.com/robots.txt"); echo $robots["body"];'
```

**Common robots.txt Issues:**
```robots
# ❌ BAD - Blocks everything
User-agent: *
Disallow: /

# ❌ BAD - Blocks important directories
User-agent: *
Disallow: /wp-content/
Disallow: /wp-admin/
Disallow: /wp-includes/

# ✓ CORRECT - Proper WordPress robots.txt
User-agent: *
Disallow: /wp-admin/
Disallow: /wp-includes/
Disallow: /wp-content/plugins/
Disallow: /wp-content/cache/
Disallow: /wp-content/uploads/private/
Disallow: *?s=
Disallow: *?p=*&p=*
Disallow: */trackback/
Disallow: */feed/

Sitemap: https://worknoon.com/sitemap.xml
```

**WordPress robots.txt Setup:**
```php
// In WordPress Admin:
// Settings → Reading → Search Engines

// Or manually create wp-root/robots.txt:
User-agent: *
Disallow: /wp-admin/
Disallow: /wp-includes/
Disallow: /wp-content/plugins/
Allow: /wp-content/uploads/

Sitemap: https://worknoon.com/sitemap.xml

# Block bad bots
User-agent: MJ12bot
Disallow: /

User-agent: AhrefsBot
Disallow: /
```

**Validation:**
- [ ] robots.txt file exists at root
- [ ] No important directories blocked
- [ ] Sitemap URL included
- [ ] Syntax is valid
- [ ] Allows common Google bots (Googlebot, Googlebot-Mobile)

### Test 2.2: SSL/HTTPS Certificate Validation

**Why This Matters:**
- Google prioritizes HTTPS
- Mixed HTTPS/HTTP causes crawl issues
- Certificate errors prevent indexation

**Check SSL Status:**
```bash
# Test SSL certificate
openssl s_client -connect worknoon.com:443

# Expected: SSL cert valid, no errors

# Or use online tool:
# https://www.sslshopper.com/ssl-checker.html
# Enter: worknoon.com
```

**Common SSL Issues:**
```
Issue: "SSL_ERROR_EXPIRED_CERT_ALERT"
→ Certificate expired
→ Renew immediately (Let's Encrypt, Comodo)

Issue: "SSL_ERROR_SELF_SIGNED_CERT"
→ Self-signed certificate
→ Use trusted CA certificate

Issue: "SSL_ERROR_UNTRUSTED_ISSUER"
→ Certificate issuer not trusted
→ Verify CA is recognized

Issue: "SSL_ERROR_BAD_CERT_DOMAIN"
→ Certificate domain doesn't match
→ Certificate must match exact domain
```

**WordPress HTTPS Enforcement:**
```php
// In wp-config.php:
define( 'FORCE_SSL_ADMIN', true );
define( 'FORCE_SSL_LOGIN', true );

// Add to functions.php:
// Force HTTPS for entire site
if ( ! is_admin() ) {
    if ( $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'http' || 
         $_SERVER['REQUEST_SCHEME'] === 'http' ) {
        wp_safe_remote_post( 'https:' . str_replace( 'http:', '', 
            'http://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'] ) );
        exit;
    }
}

// Or use this in .htaccess:
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>
```

**Validation:**
- [ ] SSL certificate is valid
- [ ] Certificate domain matches
- [ ] No expired certificates
- [ ] Chain certificate is complete
- [ ] Mixed HTTP/HTTPS resolved
- [ ] Redirect from HTTP → HTTPS works

### Test 2.3: Server Accessibility & Response Codes

**Test Server Response:**
```bash
# Check homepage HTTP response
curl -I https://worknoon.com

# Expected Response:
# HTTP/2 200 OK ✓

# Common Problem Responses:
# HTTP/1.1 404 Not Found ❌
# HTTP/1.1 403 Forbidden ❌
# HTTP/1.1 500 Internal Server Error ❌
# HTTP/1.1 503 Service Unavailable ❌
# HTTP/1.1 302 Found (redirect) - check destination
```

**Test All Pages:**
```bash
# Test multiple key pages
for page in "/" "/services/" "/about/" "/blog/"; do
    echo "Testing: https://worknoon.com$page"
    curl -I "https://worknoon.com$page" | head -1
done

# Output should be HTTP 200 for all
```

**Fix Common Response Issues:**
```php
// Fix 404 errors - Verify permalinks
wp rewrite flush --hard

// Fix 500 errors - Check error logs
// Error log location: /var/log/apache2/error.log
// Or: /var/log/php-errors.log
// Or WordPress: wp-content/debug.log

// Fix 503 errors - Check server resources
// Login to hosting control panel
// Verify CPU/Memory usage
// Check if plugins are misconfigured

// Fix timeout errors (408, 504)
// Increase PHP timeout in .htaccess
<IfModule mod_php.c>
    php_value upload_max_filesize 100M
    php_value post_max_size 100M
    php_value max_input_time 300
    php_value max_execution_time 300
</IfModule>

// Or in php.ini:
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
```

**Validation:**
- [ ] All pages return HTTP 200
- [ ] No 4xx or 5xx errors
- [ ] Server responds within 3 seconds
- [ ] No timeout errors
- [ ] Redirects are 301 (permanent)

---

## Part 3: Canonical URL Audit

### What is a Canonical URL?

```html
<!-- Canonical tells Google which version to index -->
<link rel="canonical" href="https://worknoon.com/about/" />

<!-- Without canonical, multiple URLs can cause indexing issues -->

<!-- Common Duplicate URL Problems: -->
http://worknoon.com/about/
https://worknoon.com/about/
https://www.worknoon.com/about/
https://worknoon.com/about/?utm_source=email
https://worknoon.com/about/index.html
```

### Test 3.1: Implement Canonical Tags

**WordPress Implementation:**
```php
// Add to functions.php (if not using Rankmath SEO)
add_action( 'wp_head', function() {
    if ( ! is_admin() ) {
        echo '<link rel="canonical" href="' . esc_url( get_permalink() ) . '" />' . "\n";
    }
});

// Better: Use Rankmath SEO (handles automatically)
// Install plugin, enable canonical URLs in settings
// Settings → Rankmath SEO → General → Canonical URLs: ON
```

**Check Current Canonicals:**
```bash
# Extract all canonical tags from homepage
curl https://worknoon.com | grep canonical

# Expected output:
# <link rel="canonical" href="https://worknoon.com/" />

# If no canonical tag found:
# → Add one immediately

# If canonical points to different URL:
# → Fix to point to self
```

### Test 3.2: Check Self-Referential Canonicals

```html
<!-- CORRECT: Canonical points to itself -->
<!-- Page: https://worknoon.com/about/ -->
<link rel="canonical" href="https://worknoon.com/about/" />

<!-- WRONG: Canonical points to different URL -->
<!-- Page: https://worknoon.com/about/ -->
<link rel="canonical" href="https://worknoon.com/aboutus/" />
<!-- This causes indexing confusion -->
```

### Test 3.3: Canonical Chain Issues

```html
<!-- WRONG: Chained canonicals -->

<!-- Page 1: https://worknoon.com/page1 -->
<link rel="canonical" href="https://worknoon.com/page2" />

<!-- Page 2: https://worknoon.com/page2 -->
<link rel="canonical" href="https://worknoon.com/page3" />

<!-- Page 3: https://worknoon.com/page3 -->
<link rel="canonical" href="https://worknoon.com/page3" />

<!-- This creates confusion - Use direct canonicals -->

<!-- CORRECT: Direct canonicals -->
<!-- Page 1, 2, 3 all point to: -->
<link rel="canonical" href="https://worknoon.com/page3" />
```

**WordPress Validation:**
```php
// Check canonical on every page
add_action( 'wp_head', function() {
    $canonical = get_permalink();
    $current_url = ( is_ssl() ? 'https://' : 'http://' ) . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'];
    
    // Log any mismatches
    if ( $canonical !== $current_url ) {
        error_log( 'Canonical mismatch: ' . $current_url . ' -> ' . $canonical );
    }
});
```

**Validation:**
- [ ] Every page has canonical tag
- [ ] Canonical points to itself (self-referential)
- [ ] No canonical chains
- [ ] All canonicals use HTTPS
- [ ] No canonical to www vs non-www confusion

---

## Part 4: XML Sitemap Structure Audit

### What Google Expects in Sitemap

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <!-- Each URL must have: url, lastmod, changefreq, priority -->
  <url>
    <loc>https://worknoon.com/</loc>
    <lastmod>2024-01-15</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  
  <url>
    <loc>https://worknoon.com/services/</loc>
    <lastmod>2024-01-10</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.9</priority>
  </url>
</urlset>
```

### Test 4.1: Verify Sitemap Exists & Is Valid

```bash
# Check if sitemap exists
curl https://worknoon.com/sitemap.xml

# Expected: Valid XML content (not error page)

# Validate XML syntax
# Online tool: https://www.xml-sitemaps.com/validate-xml-sitemap.html
# Upload your sitemap.xml file

# Check sitemap size
# WordPress generates separate sitemaps:
# https://worknoon.com/sitemap_index.xml (main index)
# https://worknoon.com/sitemap1.xml (posts)
# https://worknoon.com/sitemap2.xml (pages)
# Etc.
```

### Test 4.2: Sitemap Completeness Issues

```php
// Check how many URLs should be in sitemap
$args = array(
    'public'   => true,
    '_builtin' => false
);
$output = 'objects';
$post_types = get_post_types( $args, $output );

// Count published posts
$posts_count = wp_count_posts( 'post' )->publish;
$pages_count = wp_count_posts( 'page' )->publish;
$products_count = wp_count_posts( 'product' )->publish ?? 0;

$total = $posts_count + $pages_count + $products_count;
// Sitemap should include all these items
```

**Common Sitemap Issues:**

```
Issue 1: Sitemap includes noindex pages
❌ <loc>https://worknoon.com/private-page/</loc>
   (This page has no-index directive)

Fix:
✓ Exclude pages with no-index from sitemap
// In Rankmath: Settings → XML Sitemaps → Exclude these posts

Issue 2: Sitemap includes duplicate content
❌ <loc>https://worknoon.com/?p=123</loc>
❌ <loc>https://worknoon.com/about/</loc>
   (Same page, both in sitemap)

Fix:
✓ Only include canonical URL
✓ Remove query string URLs

Issue 3: Sitemap URLs are outdated
❌ <lastmod>2023-01-01</lastmod>
   (Not updated in 12 months)

Fix:
✓ Enable automatic sitemap generation
✓ Set update frequency to "weekly"

Issue 4: Sitemap has broken URLs (404s)
❌ <loc>https://worknoon.com/old-deleted-page/</loc>
   (Returns 404 Not Found)

Fix:
✓ Remove deleted page URLs
✓ Update URLs for moved pages
```

### Test 4.3: Configure WordPress Sitemaps

```php
// Enable in WordPress (Settings → Reading)
// Or programmatically:

add_filter( 'wp_sitemaps_enabled', function() {
    return true;
});

// Set WordPress post types to include
add_filter( 'wp_sitemaps_post_types', function( $post_types ) {
    // Include posts, pages, services
    return array( 'post', 'page', 'services' );
});

// Exclude posts with no-index
add_filter( 'wp_sitemaps_posts_query_args', function( $args ) {
    // Only include published, indexed posts
    $args['meta_query'] = array(
        array(
            'key' => '_Rankmath_meta-robots-noindex',
            'compare' => 'NOT EXISTS'
        )
    );
    return $args;
});

// Or use Rankmath SEO (handles automatically):
// Install Rankmath SEO plugin
// Go to SEO → XML Sitemaps
// Enable sitemaps for post types needed
```

**Validation:**
- [ ] Sitemap exists and is valid XML
- [ ] All URLs in sitemap return HTTP 200
- [ ] Sitemap only includes indexed pages
- [ ] No duplicate URLs in sitemap
- [ ] lastmod dates are current
- [ ] Sitemap submitted to Google Search Console
- [ ] No orphaned pages missing from sitemap

---

## Part 5: No-Index & Meta Tags Audit

### Test 5.1: Check for Accidental No-Index

```bash
# Check if homepage has no-index
curl -s https://worknoon.com | grep -i "no-index"

# Should return NOTHING (no no-index)

# If you see:
# <meta name="robots" content="noindex, nofollow">
# → This BLOCKS indexation (CRITICAL ISSUE)
```

**Remove No-Index if Present:**

```html
<!-- WRONG - Blocks indexing -->
<meta name="robots" content="noindex, nofollow" />

<!-- CORRECT - Allows indexing -->
<meta name="robots" content="index, follow" />
```

**WordPress Fix:**
```php
// In wp-admin -> Settings -> Reading -> Search Engine Visibility
// Checkbox: "Discourage search engines from indexing this site"
// ✓ UNCHECK THIS (should be unchecked for new sites)

// Or programmatically:
update_option( 'blog_public', '1' );
// 1 = Allow search engines
// 0 = Discourage search engines (no-index)
```

**Plugin Level No-Index:**
```php
// Check if Rankmath SEO has no-index enabled:
// SEO → Titles & Metas → Visibility
// Check each post type: "Post type visibility" should be "Show in search"

// Code to verify:
$show_posts = get_option( 'rankmath_post_type_visibility' );
if ( isset( $show_posts['post'] ) && $show_posts['post'] === 'private' ) {
    // Posts are set to no-index - Fix this
    $show_posts['post'] = 'public';
    update_option( 'rankmath_post_type_visibility', $show_posts );
}
```

### Test 5.2: Check for Hidden Text or Hidden Content

```bash
# Check page source for hidden text
curl -s https://worknoon.com | grep -E 'display:none|visibility:hidden|width:0|height:0'

# Google penalizes hidden keyword-stuffing
# Some hidden elements are OK (form labels, screen-reader only text)

# But if you see lots of hidden text with keywords:
# → Remove it (violates Google's Helpful Content Update)
```

**Legitimate Hidden Content:**

```html
<!-- OK: Screen reader only text -->
<span class="sr-only">Skip to content</span>

<!-- OK: Accordion/expandable content -->
<div style="display:none" id="accordion-content">
    Hidden until user clicks
</div>

<!-- NOT OK: Hidden keyword stuffing -->
<div style="display:none; color:white">
    wordpress seo optimization wordpress seo...
</div>
```

**Validation:**
- [ ] No-index directive is NOT present on homepage
- [ ] WordPress "Discourage search engines" is UNCHECKED
- [ ] No excessive hidden text with keywords
- [ ] All content is visible to users
- [ ] No cloaking (different content for Google vs users)

---

## Part 6: Page Speed Indexing Blockers

### Why Page Speed Matters for Indexing

Google uses "Chromium" browser to render pages. If page is too slow:
- Timeout during rendering
- Incomplete page indexing
- Core Web Vitals issues

### Test 6.1: Run Lighthouse Audit

```bash
# Using Chrome DevTools:
# 1. Open https://worknoon.com
# 2. Press F12 (Open DevTools)
# 3. Go to "Lighthouse" tab
# 4. Click "Analyze page load"

# Look for:
- Performance score: 50+ (Green)
- First Contentful Paint: < 1.8s
- Largest Contentful Paint: < 2.5s
- Cumulative Layout Shift: < 0.1
```

**Online Tools:**
```
PageSpeed Insights: https://pagespeed.web.dev/
- Enter: https://worknoon.com
- Check Mobile & Desktop scores
- Review "Opportunities" section

GTmetrix: https://gtmetrix.com/
- Detailed performance waterfall
- Specific optimization recommendations

WebPageTest: https://www.webpagetest.org/
- Advanced rendering analysis
- Video playback of page load
```

### Test 6.2: Fix Common Performance Issues

```php
// 1. Enable compression
add_action( 'init', function() {
    ob_start( 'ob_gzhandler' );
});

// 2. Enable browser caching
// Add to .htaccess:
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/html "access plus 1 hour"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType text/javascript "access plus 1 year"
</IfModule>

// 3. Lazy load images
add_filter( 'wp_get_attachment_image_attributes', function( $atts ) {
    $atts['loading'] = 'lazy';
    return $atts;
});

// 4. Minify CSS/JS
// Use plugins: WP Rocket, WP Super Cache, Autoptimize

// 5. Implement a CDN
// CloudFlare (free tier available)
// BunnyCDN, Akamai, CloudFront
```

**WordPress Performance Plugins:**
```
Recommended Setup:
1. WP Super Cache (free caching)
2. Autoptimize (free minification)
3. Smush (free image optimization)
4. CloudFlare (free CDN)
5. LiteSpeed Cache (if on LiteSpeed server)
```

**Configuration:**

```php
// WP Super Cache settings
define( 'WP_CACHE', true );

// Add to wp-config.php:
// Enable cache preloading for faster indexing
define( 'CACHE_TIME_INTERVAL', 3600 ); // Cache for 1 hour
define( 'CACHE_TIME_PERIOD', 'HOURS_1' );

// Autoptimize settings (via plugin settings):
// ✓ Minify HTML
// ✓ Minify CSS
// ✓ Minify JavaScript
// ✓ Lazy load images
// ✓ Remove Google Fonts (if not used)
```

**Validation:**
- [ ] Lighthouse Performance: 50+
- [ ] First Contentful Paint: < 1.8s
- [ ] Largest Contentful Paint: < 2.5s
- [ ] Cumulative Layout Shift: < 0.1
- [ ] Mobile performance optimized
- [ ] No render-blocking resources
- [ ] Images optimized for web

---

## Part 7: Search Console Debugging Steps

### Step 7.1: Submit Sitemap

```
In Google Search Console:

1. Click "Sitemaps" in left menu
2. Enter sitemap URL: https://worknoon.com/sitemap.xml
3. Click "Submit"
4. Wait for Google to process

Expected Result:
- Status shows "Success"
- Shows "URLs in sitemap: X"
- Shows "URLs submitted" vs "URLs indexed"
```

### Step 7.2: Request URL Indexing

```
For pages not indexed:

1. Click "URL Inspection" (top search bar)
2. Enter: https://worknoon.com/services/
3. Click "Test live URL"
4. If successful, click "Request Indexing"
5. Google will crawl and index within days

Check status in Search Console:
- "URL is on Google" = Indexed ✓
- "Crawled - Not Indexed" = Issue found ⚠
- "Not on Google" = Not crawled yet
```

### Step 7.3: Check Coverage Report Issues

```
In Search Console → Coverage:

Click on each issue type to see details:

1. ERROR (Fix immediately):
   - Server errors
   - Not found (404)
   - Access denied (401/403)
   
   Action:
   - Fix error on server
   - Ensure pages return 200 OK
   - Check that pages are live

2. VALID WITH WARNINGS (Review):
   - Duplicate without canonical
   - Excluded by no-index
   - Covered by other page
   
   Action:
   - Add canonical tags
   - Remove no-index if unintended
   - Update internal linking

3. EXCLUDED (Review):
   - Robots.txt blocking
   - Submitted URL not found
   - Soft 404
   
   Action:
   - Update robots.txt
   - Fix broken URLs
   - Check redirect chains
```

### Step 7.4: Review Search Statistics

```
Search Console → Performance:

Check:
- Impressions: How often site appears in search
- Clicks: How often clicked
- Average Position: Where ranking (#1-100)
- CTR: Click-through rate

Expected for New Site:
- Week 1-2: 0-100 impressions
- Week 2-4: 100-1000 impressions
- Month 1: 1000+ impressions
- If trending down: Indexing issues likely
```

---

## Part 8: Complete Troubleshooting Checklist

### Phase 1: Immediate Diagnostics (Day 1)

```
Crawlability:
☐ Site:worknoon.com shows results in Google
☐ Homepage in URL Inspection shows "On Google"
☐ Sitemap submitted to Search Console
☐ robots.txt allows crawling
☐ SSL certificate is valid
☐ No 404 errors on key pages

On-Page Issues:
☐ No no-index directive present
☐ Canonical tags implemented
☐ All pages return HTTP 200
☐ No excessive redirects

Technical Setup:
☐ Google Business Profile verified
☐ Schema markup implemented and valid
☐ Google Analytics 4 configured
☐ Google Search Console connected
```

### Phase 2: Content & Structure Audit (Days 2-3)

```
Content Quality:
☐ Homepage has 300+ words
☐ All pages have descriptive titles (50-60 chars)
☐ Meta descriptions present (150-160 chars)
☐ At least 10 pages on site
☐ Internal linking strategy implemented
☐ Images have alt text

Sitemap Issues:
☐ Sitemap contains all key pages
☐ URLs in sitemap return 200 OK
☐ No noindex pages in sitemap
☐ lastmod dates are recent
☐ Sitemap resubmitted if updated
```

### Phase 3: Performance & Speed (Days 4-5)

```
Page Speed:
☐ Lighthouse score 50+
☐ First Contentful Paint < 1.8s
☐ Largest Contentful Paint < 2.5s
☐ Cumulative Layout Shift < 0.1
☐ Mobile optimized (responsive design)
☐ Images optimized (WebP, compressed)
☐ CSS/JS minified and compressed

Caching:
☐ Browser caching enabled
☐ Server-side caching configured
☐ CDN enabled (CloudFlare minimum)
☐ Gzip compression enabled
```

### Phase 4: Authority & Trust (Days 6-7)

```
Authority Signals:
☐ Backlinks from related sites (10+)
☐ Social media profiles linked
☐ About page with team info
☐ Contact information present
☐ Press mentions or media coverage
☐ Customer testimonials/reviews
☐ Business registration verified

Brand Signals:
☐ Consistent logo across web
☐ Consistent company name
☐ Consistent contact information
☐ Professional design
☐ No spelling/grammar errors
```

---

## Part 9: Expected Timeline to Indexing

### New Site Timeline

```
Week 1:
- Submit sitemap
- Install Google Analytics
- Set up Search Console
- Expected indexed pages: 0-10%

Week 2:
- Monitor crawl activity
- Fix any errors found
- Build first backlinks
- Expected indexed pages: 10-30%

Week 3-4:
- Publish more content
- Outreach for links
- Build authority signals
- Expected indexed pages: 30-70%

Month 2:
- Core pages indexed
- Organic traffic begins
- Ranking for branded terms
- Expected indexed pages: 70-100%

Month 3+:
- Full indexing
- Ranking for key terms
- Sustained organic traffic
- Optimization phase
```

### Emergency Acceleration

If indexing takes > 30 days, escalate:

```
Option 1: Manual Request
1. Go to Search Console → URL Inspection
2. Enter key pages one by one
3. Click "Request Indexing"
4. Submit up to 50 pages

Option 2: Google Search Console Help
1. Search Console → Help & Feedback
2. Submit issue report with:
   - Domain
   - Expected URLs
   - Screenshots of Search Console coverage
   - What you've tried already

Option 3: Re-crawl Request
1. Search Console → Settings
2. Click "Fetch as Google"
3. Select "Desktop" or "Mobile"
4. Request Google to re-crawl
```

---

## Part 10: Prevention & Maintenance

### Monthly Checklist

```
Every Month:
☐ Check Search Console coverage report
☐ Monitor Core Web Vitals
☐ Review page performance in Analytics
☐ Check for crawl errors
☐ Verify no accidental no-index added
☐ Update sitemap if new content added
☐ Review backlink profile
☐ Test URL inspection on new pages
```

### Red Flags to Watch

```
⚠ If you see these, investigate immediately:

- Sudden drop in indexed pages
  → Check for site hacks
  → Verify no no-index added
  → Check for malware

- "Excluded" pages increasing
  → Robots.txt may have changed
  → Check no-index directives
  → Verify canonical URLs

- Crawl errors suddenly appearing
  → Server issues
  → Hosting problems
  → Plugin conflicts

- Drop in impressions/clicks
  → Rankings dropped (algorithm update)
  → Indexing issues returned
  → Content quality concerns
  → Increased competition
```