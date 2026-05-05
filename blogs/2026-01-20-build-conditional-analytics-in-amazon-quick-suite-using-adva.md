---
title: "Build conditional analytics in Amazon Quick Suite using advanced filtering"
url: "https://aws.amazon.com/blogs/business-intelligence/build-conditional-analytics-in-amazon-quick-suite-using-advanced-filtering/"
date: "Tue, 20 Jan 2026 15:55:52 +0000"
author: "Ying Wang"
feed_url: "https://aws.amazon.com/blogs/business-intelligence/category/analytics/amazon-quicksight/feed/"
---
<p><a href="https://aws.amazon.com/quicksuite/quicksight/" rel="noopener noreferrer" target="_blank">Amazon Quick Sight</a>, the business intelligence (BI) component of <a href="https://aws.amazon.com/quicksuite/?trk=948706d9-500f-422e-84ab-56d0959f121e&amp;sc_channel=ps&amp;ef_id=CjwKCAiA0eTJBhBaEiwA-Pa-hfVD5C_69qjELWCKKoCVlptNGyAVGvnKFeOa5A0mTMGHtzEALHSwIhoCzLkQAvD_BwE:G:s&amp;s_kwcid=AL!4422!3!787179384428!e!!g!!quick%20suite!23329580070!195362137328&amp;gad_campaignid=23329580070&amp;gbraid=0AAAAADjHtp91d3-5V1axj4C2bPWHAI9sa&amp;gclid=CjwKCAiA0eTJBhBaEiwA-Pa-hfVD5C_69qjELWCKKoCVlptNGyAVGvnKFeOa5A0mTMGHtzEALHSwIhoCzLkQAvD_BwE" rel="noopener noreferrer" target="_blank">Amazon Quick Suite</a>, provides powerful tools to create interactive dashboards with capabilities like filtering, conditional analytics, and parameter-driven navigation.</p> 
<p>This blog post demonstrates how to combine filters with calculated fields for precise data segmentation, control visual displays based on context, implement parameter-based navigation across dashboards, and integrate the Quick Suite chat agent for natural language querying. It provides practical techniques that BI developers can use to build more effective and responsive dashboards based on user actions and underlying data.</p> 
<h2>Solution overview</h2> 
<p>The core of BI (data analytics and visualization) effectively presents the results of SQL queries. In this context, a filter in Quick Sight is analogous to the WHERE or HAVING clause in SQL. Filters control which subset of data is displayed in visualizations, which helps users focus on relevant information and improving performance. Filters can be applied at multiple scopes, including the dataset, all sheets of an analysis, a single sheet, or individual visuals. They also operate at different stages of evaluation, such as normal analysis filters (equivalent to WHERE), top/bottom N filters (temporary results similar to a subquery used in WHERE), or filters based on aggregated calculations (equivalent to HAVING). Filter types include categorical, numeric, date, and custom calculated filters. It’s important to understand the evaluation order, because Quick Sight processes filters in a specific sequence based on both scope and stage. Generally, dataset filters are applied first, reducing the data loaded into the analysis. Analysis filters then refine the data for the entire dashboard, followed by visual filters that affect individual visuals. Within a single visual, the different stages, such as normal analysis filters, top/bottom N filters, and aggregated calculation filters, are applied in a defined order. Cascading filters, where the output of one filter influences the available elements of another, also follow this sequence, which ensure that dependency of selection in one filter to subsequent filters. This evaluation order makes sure filters work consistently when used together.</p> 
<p>In the following sections, we show how to combine filters with calculated fields, control visual displays based on context, implement parameter-based navigation across dashboards, and integrate the Quick Suite chat agent for natural language querying.</p> 
<h2>Prerequisites</h2> 
<p>To follow along with the solution presented in this post, you must meet the following requirements:</p> 
<ul> 
 <li><strong>Quick Sight access</strong> – You must have access to Quick Sight with an Author or Author Pro role.</li> 
 <li><strong>Quick Sight edition</strong> – This solution requires Quick Sight Enterprise Edition.</li> 
 <li><strong>Sample dataset</strong> – You must have a dataset with a mix of categorical and numerical fields to follow the filtering, formatting, and parameter-based navigation examples. If you don’t have a dataset prepared, you can use the public <a href="https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results" rel="noopener noreferrer" target="_blank">Olympic Games</a> dataset.</li> 
</ul> 
<p>Make sure your environment is properly set up before proceeding.</p> 
<h2>Conditional rendering of the values of a filter or control</h2> 
<p>This section demonstrates how to construct advanced filters that extend out-of-the-box features by combining calculated fields and parameters to adapt dynamically to the data context. In many analytics use cases, we need to categorize records into different segments. However, a common challenge arises when a single record belongs to multiple segments. For example, in Olympic games, water polo is both a team sport and a water sport. In traditional filtering models, handling this kind of multi-category association often requires duplicating records, which can distort aggregations and complicate analysis. To avoid this, we designed a solution using bitwise encoding, which allows each record to be represented as a unique integer that encodes its segment memberships, without creating any duplicate rows.</p> 
<h3>Implement the sport segments</h3> 
<p>To enable dynamic segmentation of sports categories within a single analysis, each sport is assigned a 4-bit binary vector that represents its membership across multiple segments. Each bit corresponds to a specific category:</p> 
<ul> 
 <li><strong>Water Sports</strong> – For example, Water Polo, Diving, Swimming, Triathlon, Sailing</li> 
 <li><strong>Ball Games</strong> – For example, Football, Basketball, Volleyball, Water Polo, Tennis, Golf</li> 
 <li><strong>Team Sports</strong> – For example, Football, Basketball, Volleyball, Water Polo</li> 
 <li><strong>All Sports </strong>– All sports provided in the database</li> 
</ul> 
<p>If a sport belongs to a segment, the corresponding bit is set to 1; otherwise, it remains 0. This results in a bitmask representation that alleviates the need to duplicate records for multi-segment classification. The following figure shows the bit index and bit values of a 4-bit binary vector.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-1-3.png"><img alt="Educational diagram showing 4-bit binary vector structure with bit indices 3, 2, 1, 0 and corresponding bit values 1, 1, 0, 1" class="alignnone wp-image-6525 size-full" height="573" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-1-3.png" width="1467" /></a></em></p> 
<p>The following table shows the mapping between the sports and their corresponding bit indexes.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Sports</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Bit Index</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Position Value in Decimal</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Water Sports</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Bit 3</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2<sup>3</sup>=8</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Ball Games</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Bit 2</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2<sup>2</sup>=4</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Team Sports</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Bit 1</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2<sup>1</sup>=2</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">All Sports</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Bit 0</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2<sup>0</sup>=1</td> 
  </tr> 
 </tbody> 
</table> 
<p>For example: <code>Water Polo</code> belongs to all four segments and is represented as <code>1111</code> (15 in decimal), while <code>Golf</code>, which belongs only to <code>All Sports</code> and <code>Ball Games</code>, is <code>0101</code> (5 in decimal).</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Sports</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Binary</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Decimal</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Explanation</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Water Polo</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-code">1111</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">15</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Belongs to all four segments</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Golf</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-code">0101</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">5</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Belongs only to <code>All Sports</code> and <code>Ball Games</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Football</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-code">0111</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">7</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Belongs only to <code>All Sports</code>, <code>Ball Games</code>, and <code>Team Sports</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Swimming</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"> 
    <div class="hide-language"> 
     <pre><code class="lang-code">1001</code></pre> 
    </div> </td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">9</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Belongs only to <code>All Sports</code> and <code>Water Sports</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">…</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"></td> 
  </tr> 
 </tbody> 
</table> 
<p>This compact bitwise representation makes it possible to encode multiple segment relationships in a single field. To build this logic in Quick Sight, complete the following steps:</p> 
<ol> 
 <li>Create a calculated field called <code>Sport Segment</code> using a bitmask formula. Each segment is assigned a bit position: 1 for <code>All Sports</code>, 2 for <code>Team Sports</code>, 4 for <code>Ball Games</code>, and 8 for <code>Water Sports</code>. Using nested <code>ifelse</code> statements, we calculate the final integer value per sport based on which segments it belongs to.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-code">Sport Segment =
1+ 2 * ifelse(sport = 'Football' or sport = 'Basketball' or sport = 'Volleyball' or sport = 'Water Polo', 1, 0)+ 4 * ifelse(sport = 'Football' or sport = 'Basketball' or sport = 'Volleyball' or sport = 'Water Polo' or sport = 'Tennis' or sport = 'Golf', 1, 0)+ 8 * ifelse(sport = 'Water Polo' or sport = 'Diving' or sport = 'Swimming' or sport = 'Sailing' or sport = 'Triathlon', 1, 0)</code></pre> 
</div> 
<ol start="2"> 
 <li>Create a parameter named <code>SegmentSelection</code> with values like <code>All Sports</code>, <code>Team Sports</code>, <code>Ball Games</code>, and <code>Water Sports</code>. Users can choose a segment from a dropdown menu.</li> 
 <li>To apply this segmentation interactively, define another calculated field called <code>segmentChoice</code> that maps each segment name to its corresponding bit value (for example, <code>Ball Games</code> = 4):</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">segmentChoice =&nbsp;
ifelse(${SegmentSelection} = 'All Sports', 1,&nbsp;${SegmentSelection} = 'Team Sports', 2,&nbsp;${SegmentSelection} = 'Ball Games', 4,&nbsp;${SegmentSelection} = 'Water Sports', 8,&nbsp;NULL)</code></pre> 
</div> 
<ol start="4"> 
 <li>Compute a <code>segmentIndicator</code> using a bitwise check: divide the sport’s <code>Sport Segment</code> value by the selected <code>segmentChoice</code>, take the floor, and apply <code>mod 2</code> to extract the bit at the corresponding position. Create the calculated field <code>segmentIndicator</code> =<code>mod(floor({Sport Segment}/segmentChoice), 2)</code>. A filter is applied to show only records where <code>segmentIndicator = 1</code>, meaning the sport belongs to the selected segment. This method supports fast, scalable filtering logic using binary arithmetic, without complex joins or lookup tables.</li> 
</ol> 
<p>The following screenshot shows the results when a user chooses <strong>Ball Games</strong> from the control.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-2-4.png"><img alt="Sport Segment Selection displaying Ball Games category with six sports including Basketball, Football, Golf, Tennis, Volleyball, and Water Polo" class="alignnone wp-image-6524 size-full" height="552" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-2-4.png" width="648" /></a></em></p> 
<p>The following screenshot shows the results when a user chooses <strong>Team Sports</strong> from the control.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-3-5.png"><img alt="Sport Segment Selection showing Team Sports category with Basketball, Football, Volleyball, and Water Polo statistics" class="alignnone wp-image-6523 size-full" height="460" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-3-5.png" width="676" /></a></em></p> 
<p>The following screenshot shows the results when a user chooses <strong>Water Sports</strong> from the control.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-4-3.png"><img alt="Sport Segment Selection control showing Water Sports category with filtered table of five water-based sports and their team counts" class="alignnone wp-image-6522 size-full" height="514" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-4-3.png" width="696" /></a></em></p> 
<p>Open the <a href="https://6d4u42bbb5.execute-api.us-east-1.amazonaws.com/prod/sample-dashboard-filter" rel="noopener noreferrer" target="_blank">sample dashboard</a> and go to the “Segmentation” sheet.</p> 
<h3>Filter for A AND B in Quick Sight</h3> 
<p>In Quick Sight, applying a multi-value filter typically operates as an OR condition, returning results where a field matches any of the selected values. For example, if you choose <strong>Archery</strong> and <strong>Basketball</strong> as the filter on <strong>Sports</strong>, Quick Sight will return all countries that play either sport. However, in some use cases, we need to identify records that meet all selected conditions simultaneously, such as finding countries or teams that participate in both archery and basketball. This A AND B filtering logic isn’t supported natively, but you can achieve it using calculated fields.</p> 
<p>As shown in the following screenshot, nine countries participate in archery.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-5.jpeg"><img alt="Sports filter displaying Archery results showing nine countries with one record each" class="alignnone wp-image-6521 size-full" height="822" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-5.jpeg" width="642" /></a></em></p> 
<p>As shown in the following screenshot, three countries participate in basketball.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-6-2.jpeg"><img alt="Sports filter set to Basketball showing three countries: France, Japan, and United States of America, each with count of 1" class="alignnone wp-image-6520 size-full" height="530" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-6-2.jpeg" width="648" /></a></em></p> 
<p>If you choose both <strong>Archery</strong> and <strong>Basketball</strong> in the <strong>Sports</strong> filter, Quick Sight will return 11 countries—because it applies an OR condition by default, showing all countries that play either sport.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-7-1.jpeg"><img alt="Sports dropdown showing Basketball and Archery with filtered results displaying multiple countries and their counts" class="alignnone wp-image-6519 size-full" height="944" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-7-1.jpeg" width="650" /></a></em></p> 
<p>However, what we want is to return only the country (Japan) that participates in both sports: archery and basketball.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-8-1.jpeg"><img alt="QuickSight filter control showing Sports dropdown with Archery and Basketball selected, displaying Japan in Country table below" class="alignnone wp-image-6518 size-full" height="380" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-8-1.jpeg" width="536" /></a></em></p> 
<p>In the following steps, we walk through implementing AND-based filtering logic in Quick Sight, returning only the records that match all selected values:</p> 
<ol> 
 <li>Create a calculated field called <code>Count Sports</code> that calculates the total number of sports selected in the filter, using the expression <code>DistinctCountOver(Sports, [], PRE_AGG)</code>.</li> 
 <li>Create another calculated field called <code>Count Sports per Country</code> that determines how many of the selected sports are associated with each country, with the expression <code>DistinctCountOver(Sports, [Country], PRE_AGG)</code>.</li> 
 <li>Define a third calculated field named <code>New Country</code> that returns the country name only if it is associated with all the selected sports—otherwise, it outputs <code>N/A</code>—using the expression <code>ifelse(Count Sports = Count Sports per Country, Country, 'N/A')</code>.</li> 
 <li>In the visual, use <code>"Sports"</code> as a filter and group by <code>"New Country"</code>. Add an additional filter to exclude the <code>'N/A'</code> values from the <code>"New Country"</code> field.</li> 
</ol> 
<p>As a result, when a user selects multiple sports, the visual will display only those countries that participate in all selected sports. This effectively transforms the default OR logic into an AND logic, enabling intersection-style analysis without requiring any changes to your data model.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-9-2.jpeg"><img alt="Amazon QuickSight Filters panel showing 29 filters applied, with New Country filter set to Does not equal N/A scoped to only this visual" class="alignnone wp-image-6517 size-full" height="376" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-9-2.jpeg" width="474" /></a></em></p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-10-1.jpeg"><img alt="Sports filter dropdown showing Archery and Basketball as selected values in collapsed state" class="alignnone wp-image-6516 size-full" height="218" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-10-1.jpeg" width="530" /></a></em></p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-11-1.jpeg"><img alt="Filtered table view showing Japan when Basketball and Archery sports are selected from dropdown" class="alignnone wp-image-6515 size-full" height="414" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-11-1.jpeg" width="532" /></a></em></p> 
<p>In the visual formatting, rename the title “New Country” to be “Country.”</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-12-1.jpeg"><img alt="Amazon QuickSight Properties panel showing pivot table visual settings with Group-by column names section expanded, displaying Country field configuration" class="alignnone wp-image-6514 size-full" height="419" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-12-1.jpeg" width="241" /></a></em></p> 
<p>Open the <a href="https://6d4u42bbb5.execute-api.us-east-1.amazonaws.com/prod/sample-dashboard-filter" rel="noopener noreferrer" target="_blank">sample dashboard</a> and go to the “A AND B” sheet.</p> 
<h2>Build multiple sheets dashboards with context-aware navigation</h2> 
<p>The purpose of this section is to help BI authors link navigation actions to specific sheets based on the content of visuals, using predefined conditions. By default, an author can create only one “Select actions” configuration per visual.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-13-1.jpeg"><img alt="Amazon QuickSight new action configuration panel showing Navigation action setup with Action 2 name and menu option activation selected" class="alignnone wp-image-6513 size-full" height="951" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-13-1.jpeg" width="363" /></a></em></p> 
<p>To address this constraint, we defined a solution that enables navigation to different sheets or external webpages based on categorical values, which is triggered by a single-click “Select action.” This is achieved by using a conditional navigation action navigate to different sheets of the same dashboard. This approach provides a scalable way to build dashboards with context-aware navigations. Complete the following steps:</p> 
<ol> 
 <li>Identify the sheet IDs: 
  <ol type="a"> 
   <li>Publish the analysis to a dashboard to be able to get the ID of the dashboard.</li> 
   <li>Navigate to the web address to identify the dashboard ID.</li> 
   <li>Copy the address and note it down for later. The address should be <code>https://</code>&lt;&lt;region name&gt;&gt;<code>.quicksight.aws.amazon.com/sn/account/</code>&lt;&lt;account name&gt;&gt;<code>/dashboards/</code>&lt;&lt;dashboard ID&gt;&gt;.</li> 
  </ol> </li> 
 <li>To create the navigation action, create a “Select action” with the URL action type.</li> 
 <li>Identify the sheet ID for the two different sheets in the dashboard.</li> 
</ol> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-14-2.png"><img alt="Amazon QuickSight pivot table showing Olympic medals won by USA, Canada, and Mexico across multiple years and sports categories" class="alignnone wp-image-6512 size-full" height="340" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-14-2.png" width="1432" /></a></em></p> 
<ol start="4"> 
 <li>Now that you have the dashboard URL and the sheet ID for both sheets, you can create a calculated field for navigating to the two sheets. Here, use an <code>ifelse</code> statement to return the value of the sheet ID for the first sheet; otherwise, return the value of the sheet ID for the second sheet:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-shell">ifelse(CountryAbbrev='USA' or CountryAbbrev='CAN' or CountryAbbrev='MEX', 
&lt;&lt;USA, CAN, MEX Sheet ID&gt;&gt;, &lt;&lt;Other Countries Sheet ID&gt;&gt;)</code></pre> 
</div> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-15-2.png"><img alt="Amazon QuickSight calculated field editor showing ConditionalLinkDrillDown formula with ifelse logic for country-based URL navigation" class="alignnone wp-image-6511 size-full" height="413" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-15-2.png" width="1432" /></a></em></p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-16-2.png"><img alt="Amazon QuickSight pivot table showing comprehensive Olympic medals data across all countries, years from 1896-1948 visible, organized by year and country" class="alignnone wp-image-6510 size-full" height="698" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-16-2.png" width="1430" /></a></em></p> 
<ol> 
 <li>In the pivot table visual, insert the calculated field you created called <code>ConditionalLinkDrillDown</code> and then hide this field.</li> 
 <li>To ensure that when the user clicks on the data field in the “All Medals” sheet that the correct filters are applied to other sheets, you must pass in the parameters from the first sheet.</li> 
</ol> 
<p>The following screenshots illustrate the filters for the various sheets:</p> 
<ul> 
 <li>“USA, CAN, MEX” sheet</li> 
</ul> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-17-1.png"><img alt="Amazon QuickSight pivot table displaying Olympic medals for USA, Canada, and Mexico from 1896 through 1948 across multiple sports" class="alignnone wp-image-6509 size-full" height="691" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-17-1.png" width="1432" /></a></em></p> 
<ul> 
 <li>Other countries sheet</li> 
</ul> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-18-2.png"><img alt="Amazon QuickSight pivot table displaying Olympic medals by country and sport, filtered to show countries other than USA, Canada, and Mexico" class="alignnone wp-image-6508 size-full" height="702" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-18-2.png" width="1430" /></a></em></p> 
<ol> 
 <li>After identifying the necessary parameters, you need to pass in the parameters after the &lt;&lt;ConditionalLinkDrillDown&gt;&gt; part of the URL. Here, enter <code>#</code> followed by <code>p.parameter =</code>&lt;&lt;field associated with the parameter&gt;&gt;. When adding multiple parameters and when using multiple values for the same parameter, use the delimiter &amp;.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-shell">https://us-east-1.quicksight.aws.amazon.com/sn/account/&lt;&lt;account name&gt;&gt;/dashboards/&lt;&lt;dashboard id&gt;&gt;/sheets/&lt;&lt;ConditionalLinkDrillDown&gt;&gt;#p.pCountryAbbrev=&lt;&lt;CountryAbbrev&gt;&gt;&amp;p.pSport=&lt;&lt;sport&gt;&gt;&amp;p.pYear=&lt;&lt;Year&gt;&gt;</code></pre> 
</div> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-19-1.jpeg"><img alt="Amazon QuickSight dashboard showing Metrics Summary table with sales data by region and country, including revenue and quantity metrics across Strategic, SMB, and Enterprise segments" class="alignnone wp-image-6507 size-full" height="612" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-19-1.jpeg" width="1289" /></a></em></p> 
<p>Now, when a user clicks on a record in the “All Medals” sheet, they will be directed to the “USA, CAN, MEX” sheet if the record is associated with those three countries, and will be directed to the other countries sheet if the record is associated with countries other than USA, Canada, or Mexico.</p> 
<h2>Use Quick Suite topics with custom instructions to include or exclude specific data</h2> 
<p><strong>Quick Suite topics</strong> are collections of one or more datasets that define a specific subject area your business users can explore and ask questions about. However, ambiguity in business terminology can lead to inaccurate results if the model misinterprets user intent.</p> 
<p>In sales scenarios, two examples of such ambiguity are:</p> 
<ul> 
 <li><strong>VIP</strong> – Refers specifically to top tier customers managed by the sales team</li> 
 <li><strong>Presidential election impact</strong> – Refers only to activity during January 2024 to December 2024</li> 
</ul> 
<p>These aren’t always encoded directly in the data model, yet we still want the topic to understand them consistently. This is where custom instructions come in: authors can guide how Quick Sight interprets certain terms, even if the underlying dataset doesn’t contain explicit filters or tags for those concepts. Consider typical sales-oriented queries:</p> 
<ul> 
 <li>“Show VIP sales?”</li> 
 <li>“Sales during presidential election?”</li> 
</ul> 
<p>Without additional logic, Quick Sight can’t interpret “VIP.” Similarly, the system might not know which election you’re referencing, or which date range is relevant. Even when the dataset doesn’t explicitly define these concepts, custom instructions help authors enforce business-specific interpretations. To add custom instructions, complete the following steps:</p> 
<ol> 
 <li>On the Quick Sight console, go to the topic associated with your sales dataset.</li> 
 <li>On the <strong>Custom Instructions</strong> tab, add the following:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-code">When users refer to "VIP", interpret this as customers where 'Atlas Assured'.</code></pre> 
</div> 
<p>This facilitates consistent filtering, even if “VIP” isn’t a literal field or category in the dataset.</p> 
<ol start="3"> 
 <li>Next, add another instruction for the election impact time window:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-code">When users refer to “presidential election impact,” restrict analysis to dates between January 1, 2024 and December 31, 2024.</code></pre> 
</div> 
<p>This avoids incorrect assumptions about historical elections or post-2024 timeframes.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-20-1.jpeg"><img alt="Amazon Q Business topic configuration showing Custom Instructions tab with field definitions and context settings for Software Sales topic" class="alignnone wp-image-6506 size-full" height="648" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-20-1.jpeg" width="2560" /></a></em></p> 
<p>These instructions act like invisible filters or transformers applied at query time. Let’s explore how Quick Sight processes a few example questions after these rules are in place, as illustrated in the following table.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">User Question</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Behind-the-Scenes Logic</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">“Show VIP sales”</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Adds filter: <code>Customer = Atlas Assured</code></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">“Sales during presidential election?”</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Adds filter: <code>CloseDate BETWEEN '2024-01-01' AND '2024-12-31'</code></td> 
  </tr> 
 </tbody> 
</table> 
<p>Chat agents integrate with topics to deliver business-aware analytics through large language models. When you connect custom instructions to topics like software sales, the chat agent automatically applies your predefined business rules during every query interaction.</p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-21-2.png"><img alt="Amazon Q Business dashboard showing VIP Customer Sales by Region bar chart with Atlas Assured customers, EMEA leading at approximately $1,200 in sales" class="alignnone wp-image-6505 size-full" height="806" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-21-2.png" width="1429" /></a></em></p> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-22-2.png"><img alt="Amazon Q Business interface showing bar chart of sales by product during 2024 Presidential election year, with Site Analytics leading at $2,310" class="alignnone wp-image-6504 size-full" height="806" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/09/image-22-2.png" width="1429" /></a></em></p> 
<p>Even if your dataset doesn’t have a literal <code>VIP</code> column or a <code>presidential election</code> flag, these custom rules bring in business-aware logic without modifying the underlying data model.</p> 
<h2>Clean up</h2> 
<p>To avoid incurring ongoing charges after completing the tutorial, delete the resources you created during the walkthrough:</p> 
<ol> 
 <li>Delete any sample datasets or custom data sources that are no longer needed.</li> 
 <li>If your dataset was pulled from <a href="https://aws.amazon.com/pm/serv-s3/?trk=59968c08-333e-424a-b5a8-4fd08af5af4d&amp;sc_channel=ps&amp;ef_id=CjwKCAiA0eTJBhBaEiwA-Pa-hVgvA-VySdIL6w_z-CRo4OGyt969P0z63JpD1yRFWnKnSGVvXZ1MRBoCM4EQAvD_BwE:G:s&amp;s_kwcid=AL!4422!3!651751060962!e!!g!!amazon%20s3!19852662362!145019251177&amp;gad_campaignid=19852662362&amp;gbraid=0AAAAADjHtp-40mUVK6CgcO2H1YkKAgVYf&amp;gclid=CjwKCAiA0eTJBhBaEiwA-Pa-hVgvA-VySdIL6w_z-CRo4OGyt969P0z63JpD1yRFWnKnSGVvXZ1MRBoCM4EQAvD_BwE" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) or connected to other AWS services, be sure to delete any temporary files, tables, or queries created for this walkthrough.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>Quick Sight offers flexible ways to enrich data analysis by combining natural language, data modeling, and custom filtering logic. With custom instructions in Quick Sight, teams can enhance user experience by embedding domain-specific knowledge directly into natural language queries, providing accurate and business-aligned responses. At the data modeling layer, techniques like bitwise encoding enable efficient representation of many-to-many classifications without requiring any additional reference tables, simplifying filters, preserving data integrity, and supporting segmentation through parameters and calculated fields. Finally, by extending the default filtering behavior of Quick Sight, users can implement AND-based multi-select filters to isolate results that meet all selected conditions, delivering deeper insights without altering the dataset. Together, these approaches empower organizations to build scalable, precise, and context-aware dashboards that align closely with business needs.Try out these features for your own use case, and share your feedback in the comments.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-6535 size-thumbnail alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/13/Picture1-100x100.jpg" width="100" /><strong> Ying Wang </strong>is a Senior Specialist Solutions Architect in the Generative AI organization at AWS, specializing in Amazon Quick Suite to support large enterprise, ISV and public sector customers. She brings 16 years of experience in data analytics and data science, with a strong background as a data architect and software development engineering manager. As a data architect, Ying helped customers design and scale enterprise data architecture solutions in the cloud. In her role as an engineering manager, she enabled customers to unlock the power of their data through Quick Suite analytical engine by delivering new features and driving product innovation from both engineering and product perspectives.</p> 
<p style="clear: both;"><img alt="" class="size-thumbnail wp-image-6528 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/12/image1-100x100.png" width="100" /><strong> Michael Wong</strong> is an Associate Delivery Consultant in the AWS Professional Services organization, specializing in Data Analytics to support enterprise and public sector customers. He brings extensive experience in cloud data architecture and analytics, with a strong background in data visualization and business intelligence solutions. As a data analytics specialist, Michael has helped customers design and implement scalable data lake architectures and migrate legacy reporting systems to modern cloud-based solutions. In his consulting role, he has delivered complex Amazon Quick Sight dashboards, developed data transformation pipelines using AWS Glue and SQL, and provided technical leadership on multi-faceted data integration projects across federal and state government agencies.</p> 
<p style="clear: both;"><em><img alt="" class="size-thumbnail wp-image-6532 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/12/image-3_imresizer1-e1768254647702-100x100.jpg" width="100" /></em><strong> Priya Kakarla</strong> is a Specialist Solutions Architect for Amazon Quick Suite, with experience spanning healthcare, finance, and digital-native businesses. She helps customers design and implement end-to-end Analytics solutions that drive data-informed decision-making. Passionate about empowering organizations through intuitive and scalable analytics, Priya is known for her strong customer obsession and commitment to delivering innovative, personalized solutions that achieve real business outcomes. Outside of work, Priya loves exploring new places, trying different cuisines, and spending quality time with her family and friends.</p>
