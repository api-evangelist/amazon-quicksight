---
title: "Amazon Quick Sight expands dashboard customization in tables and pivot tables"
url: "https://aws.amazon.com/blogs/business-intelligence/amazon-quick-sight-expands-dashboard-customization-in-tables-and-pivot-tables/"
date: "Fri, 16 Jan 2026 21:46:58 +0000"
author: "Ying Wang"
feed_url: "https://aws.amazon.com/blogs/business-intelligence/category/analytics/amazon-quicksight/feed/"
---
<p>We are excited to announce the general availability of expanding dashboard customizations of tables and pivot tables in <a href="https://aws.amazon.com/quicksight/?utm_source=chatgpt.com" rel="noopener noreferrer" target="_blank">Amazon Quick Sight</a>, the AI-powered business intelligence (BI) service. With this enhancement, readers can add or remove fields, change aggregations, and modify formatting directly in dashboards without requiring updates from authors. This enhancement boosts self-service analytics, reducing reliance on authors, and helps readers tailor insights to their role.</p> 
<h2>Solution overview</h2> 
<p>Previously, readers had limited flexibility to modify dashboard visuals. With the <a href="https://aws.amazon.com/blogs/business-intelligence/empower-readers-with-customizable-tables-and-pivot-tables-in-amazon-quick-sight/" rel="noopener noreferrer" target="_blank">dashboard customizations release in November 2025</a>, Quick Sight readers can sort, reorder, hide, show, and freeze fields in tables and pivot tables, according to their needs. With the expanded capabilities introduced in the December 2025 release, readers can now also:</p> 
<ul> 
 <li>Add and remove fields in tables and pivot tables</li> 
 <li>Change value aggregations</li> 
 <li>Modify field formatting</li> 
</ul> 
<p>In the November release, we introduced the ability for readers to persist their customizations and share personalized views. In the December 2025 release, we continue to support these capabilities:</p> 
<ul> 
 <li>Save and persist the customizations for future sessions</li> 
 <li>Share customized views through bookmarks and shared views</li> 
 <li>Schedule and export customized visuals in PDF, CSV, and Excel formats</li> 
</ul> 
<p>These capabilities address common pain points and help readers tailor dashboards to their specific needs. For example, a financial analyst can remove irrelevant columns to declutter their view and focus solely on key performance indicators, a marketing specialist can reorder columns to better visualize campaign trends and outcomes, and an operations manager can export a customized pivot table in Excel format to share with stakeholders for streamlined decision-making.</p> 
<p>In the following sections, we demonstrate how to use these new features with a real-world use case.</p> 
<h2>Prerequisites</h2> 
<p>Before you begin, make sure you have the following:</p> 
<ul> 
 <li>An active AWS account with permissions to access Quick Sight</li> 
 <li>Quick Sight Enterprise Edition enabled in your account</li> 
 <li>At least one Quick Sight user (Author or Author Pro) to create and manage dashboards</li> 
 <li>Basic familiarity with Quick Sight concepts such as datasets, dashboards, analyses, and permissions</li> 
</ul> 
<h2>Enable reader customization</h2> 
<p>As an author, you can enable reader customization for a table or pivot table in just a few steps:</p> 
<ol> 
 <li>Open your analysis in Quick Sight.</li> 
 <li>Navigate to the table or pivot table you want to make customizable.</li> 
 <li>Choose <strong>Format visual</strong>.<br /> <img alt="" class="alignnone wp-image-6367 size-full" height="319" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/17/BI-7963-image-1-1.png" style="margin: 10px 0px 10px 0px;" width="1261" /></li> 
 <li>In the <strong>Properties</strong> pane, on the <strong>Interactions</strong> tab, turn on <strong>Reader Customization</strong>. (By default, this option is enabled. If the author doesn’t want readers to customize the visual, they can turn it off.)<br /> <img alt="" class="alignnone wp-image-6369 size-full" height="291" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/17/BI-7963-image-3-2.png" style="margin: 10px 0px 10px 0px;" width="300" /></li> 
 <li>With <strong>Reader Customization</strong> enabled, manage which fields you want readers to be able to add or remove. The available fields are based on the dataset that the visual is sourced from. By default, if the author doesn’t enable any additional fields, readers can remove, add back, hide, show, reorder, and change aggregations for the existing fields in the visual.</li> 
</ol> 
<p>For our example, the author created a table visual with a default view that includes the dimensions <code>Industry</code>, <code>Sales Domain</code>, <code>Customer Region</code>, and <code>Order Date</code>, as well as the measure <code>Revenue</code>.</p> 
<p>To make this table useful for multiple reader groups, the author enabled Reader customization and added twelve additional fields—including <code>City</code>, <code>Profit</code>, and <code>Quantity</code>, to the list of fields available for customization.</p> 
<p><img alt="" class="alignnone wp-image-6372 size-full" height="890" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/17/BI-7963-image-6-1.png" style="margin: 10px 0px 10px 0px;" width="500" /></p> 
<p>By including these more granular fields, readers can accomplish the following:</p> 
<ul> 
 <li>Inspect sales details by city, customer, or other dimensions</li> 
 <li>Compare sales revenue with profit and sales quantities to better understand performance</li> 
 <li>Tailor the view to their specific analytical needs without waiting for updates from the author</li> 
</ul> 
<p>With this configuration, readers can tailor the table or pivot table to their needs, while authors maintain control over which fields remain customizable.</p> 
<p><img class="alignnone size-full" height="720" src="https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/BI-7963/Video1_author.gif" width="1080" /></p> 
<h2>Add, remove, show, hide, reorder fields, and change field aggregations in tables and pivot tables</h2> 
<p>With reader customization enabled and with selected additional fields, readers can make changes to the table visual to fit their own analysis needs:</p> 
<ol> 
 <li>Open the dashboard and locate the table or pivot table with <strong>Reader Customization</strong> enabled.</li> 
 <li>Choose <strong>Customize</strong> on the visual.<br /> <img alt="" class="alignnone size-full wp-image-6363" height="374" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/17/BI-7963-image-7.png" style="margin: 10px 0px 10px 0px;" width="1240" /></li> 
 <li>From the field list, add fields such as <code>City</code>, <code>Profit</code>, or <code>Quantity</code>.<br /> <img alt="" class="alignnone wp-image-6373 size-full" height="429" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/12/17/BI-7963-image-8-1.png" style="margin: 10px 0px 10px 0px;" width="500" /></li> 
 <li>Change the Aggregate of <code>Order Date</code> to be Quarter.</li> 
 <li>Change the Aggregate of <code>Quantity</code> to be Average.</li> 
 <li>Reorder fields by dragging them into the desired sequence.</li> 
 <li>Save your customized view as a bookmark or share it with others. You can also export the customized view to PDF, CSV, or Excel for reporting purposes.</li> 
</ol> 
<p>With these capabilities, a reader such as a financial analyst can quickly compare <code>Revenue</code> against <code>Profit</code> and <code>Avg(Quantity)</code>, and a regional manager can drill down by <code>City</code> or <code>Customer</code> to identify local trends.</p> 
<p><img class="alignnone size-full" height="720" src="https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/BI-7963/video2_reader.gif" width="1080" /></p> 
<h2>Real-world use case</h2> 
<p>Let’s walk through how reader customization can help different business users get the insights they need without waiting for authors to make changes.</p> 
<p>For our use case, a sales performance dashboard is shared across multiple teams. The author has enabled Reader Customization and included additional fields such as <code>City</code>, <code>Customer</code>, <code>Profit</code>, and <code>Avg(Quantity)</code>. Our scenario involves the following personas:</p> 
<ul> 
 <li><strong>Financial analyst</strong> – Focused on profitability, the analyst removes columns like <code>City</code> and <code>Customer</code>, and reorders the table to place <code>Profit</code> directly beside <code>Revenue</code>. This way, they can quickly calculate margins and export the view to Excel for further modeling.</li> 
 <li><strong>Regional sales manager</strong> – Interested in performance by location, the manager shows the <code>City</code> column and filters for their region. They drill down into sales details to spot which cities are outperforming and identify areas that need attention.</li> 
 <li><strong>Operations manager</strong> – Concerned about inventory, the manager adds the <code>Sum(Quantity)</code> and <code>Avg(Quantity)</code> fields to compare sales volumes with revenue. They bookmark this customized view and schedule a recurring export to CSV to share with the supply chain team.</li> 
</ul> 
<p>By giving readers control over their own views, customizable tables and pivot tables help users run self-service analytics tailored to their role, and authors retain governance over which fields are available. Reader customizations in the reader view are not visible to other readers of the same dashboard, unless the originating reader shares the customized view. We provide details on view sharing and collaboration in the following section.</p> 
<h2>Saving and sharing customized views</h2> 
<p>Quick Sight provides multiple ways for readers to share their customized dashboards with colleagues:</p> 
<ul> 
 <li><strong>Share this view</strong> – Readers can quickly share their current customized view of a table or pivot table without creating a bookmark. With the Share this view option, they can generate a link that preserves their applied filters, column selections, and ordering. This makes it straightforward to collaborate on one-time (ad-hoc) analysis and make sure teammates are looking at the exact same view of the data in real time.</li> 
 <li><strong>Bookmarks</strong> – For recurring needs, readers can save their customizations as a bookmark. Bookmarks capture visual customizations and applied filters, so readers can return to their preferred view at any time. Bookmarks can be private for individual use or shared across teams to standardize insights. This is especially helpful for recurring reporting cycles or when different stakeholders need to work from a consistent, tailored perspective.</li> 
</ul> 
<p>Together, these sharing options give readers the flexibility to collaborate instantly or preserve and distribute their customized views for ongoing use.</p> 
<p><img class="alignnone size-full" height="1080" src="https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/BI-7963/BookMark_New.gif" width="1920" /></p> 
<h2>Limitations</h2> 
<p>At the time of writing, reader customization is supported only for tables and pivot tables. Other visual types, such as bar charts, line charts, or KPIs, do not offer reader-level customization at this time. Authors should plan their dashboards accordingly, using tables and pivot tables when they want to provide readers with the flexibility to add, remove, or reorder fields. AWS will continue to evaluate expanding reader customization to additional visual types in the future.</p> 
<h2>Embedding scenarios</h2> 
<p>When embedding customized dashboards, the behavior differs depending on the embedding mode. The following are key scenarios and how they handle customization and persistence:</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Embedding Method</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Can make reader customizations?</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Persistence</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Visual embedding (registered or anonymous users)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Yes</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>No persistence</strong>: when the page reloads, the original dashboard is displayed</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Dashboards embedding for registered users</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Yes</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Behavior on reload depends on persistence settings</strong>:<br /> If state persistence is enabled (through embedding options), the customized view is preserved and reloaded.<br /> If state persistence is not enabled, the original dashboard is displayed.</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Dashboards embedding for anonymous (unregistered) users</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Yes</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>No persistence</strong>: when the page reloads, the original dashboard is displayed</td> 
  </tr> 
 </tbody> 
</table> 
<p>In addition, the <code>createSharedView</code> SDK function supports generating a shared view from a customized dashboard, consistent with current functionality.</p> 
<h2>Conclusion</h2> 
<p>Reader customization of tables and pivot tables give readers greater control over their analytics experience in Quick Sight. By adding the option for readers to add, remove, show, hide, reorder, and change field aggregations, as well as share customized views, Quick Sight reduces reliance on authors and speeds up decision-making across teams. Readers can tailor dashboards to fit their role, such as a financial analyst comparing profit to revenue, a regional manager investigating city-level performance, or an operations manager scheduling exports for their team.</p> 
<p>For more details, refer to the <a href="https://docs.aws.amazon.com/quicksight/?utm_source=chatgpt.com" rel="noopener noreferrer" target="_blank">Amazon Quick Sight documentation</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-5230 size-thumbnail" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/07/31/ying-photo-1-100x133.jpeg" width="100" /><strong>Ying Wang</strong> is a Senior Specialist Solutions Architect in the Generative AI organization at AWS, specializing in Amazon QuickSight and Amazon Q to support large enterprise and ISV customers. She brings 16 years of experience in data analytics and data science, with a strong background as a data architect and software development engineering manager.&nbsp;As a data architect, Ying helped customers design and scale enterprise data architecture solutions in the cloud. In her role as an engineering manager, she enabled customers to unlock the power of their data through QuickSight by delivering new features and driving product innovation from both engineering and product perspectives.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-5587 size-thumbnail" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/09/22/bhatv-100x133.jpg" width="100" /><strong>Vasha Bhatari</strong>&nbsp;is a Senior Product Manager at Amazon QuickSight, where she drives solutions that simplify BI migrations and help customers modernize analytics with ease. Since joining Amazon in 2017, she has led initiatives across last-mile routing optimization, database migration, and business intelligence, bringing broad experience to complex data challenges. Outside of work, Vasha is always planning her next trip, trying new foods, and exploring the best hiking and kayaking spots across the Pacific Northwest.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-5589 size-thumbnail" height="133" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2025/09/22/baachler-100x133.jpg" width="100" /><strong>Bhakti Achlerkar</strong>&nbsp;is a Software Development Manager working on the core analytics of Amazon QuickSight. Before that, she was a Front-End Engineer focused on data visualization. With 10 years of experience in software development, Bhakti enjoys building customer-facing features that make products more useful and impactful.</p>
