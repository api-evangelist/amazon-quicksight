---
title: "SISAMEX Revolutionizes Inventory Analytics with Amazon Quick Suite"
url: "https://aws.amazon.com/blogs/business-intelligence/sisamex-revolutionizes-inventory-analytics-with-amazon-quick-suite/"
date: "Wed, 14 Jan 2026 22:21:11 +0000"
author: "Francisco Gutiérrez, Belinda Quintanilla"
feed_url: "https://aws.amazon.com/blogs/business-intelligence/category/analytics/amazon-quicksight/feed/"
---
<p>Sistemas Automotrices de México (<a href="https://www.sisamex.com.mx/home" rel="noopener noreferrer" target="_blank">SISAMEX</a>), a joint venture between Grupo Quimmco and Cummins, stands as a leading manufacturer of automotive components for commercial, agricultural, and off-highway vehicles. With over 45 years of experience and facilities in Nuevo León and Jalisco, SISAMEX serves global brands including Daimler Trucks, International, John Deere, Paccar, and Mercedes-Benz.</p> 
<p>In this post, learn how SISAMEX revamped their inventory management by implementing, <a href="https://aws.amazon.com/quicksuite/quicksight/" rel="noopener noreferrer" target="_blank">Amazon Quick Sight,</a> the business intelligence (BI) capability of <a href="https://aws.amazon.com/quicksuite/" rel="noopener noreferrer" target="_blank">Amazon Quick Suite</a>, transforming a 3-hour manual process into a 1-minute automated solution with real-time analytics and enhanced data accuracy.</p> 
<h2><strong>The challenge: Manual processes limiting growth</strong></h2> 
<p>Before implementing Quick Sight, SISAMEX faced several critical operational challenges:</p> 
<ul> 
 <li><strong>Time and efficiency issues</strong> – SISAMEX’s inventory management process was heavily manual and time intensive. Each team member spent approximately 3 hours processing inventory analysis, with an additional 3 hours needed to respond to supplier queries due to the manual nature of data handling. The process was further complicated by significant dead times in document consolidation and arrangement, as teams struggled with gathering and organizing data from various sources.</li> 
 <li><strong>Data quality and process challenges –</strong> The manual nature of the inventory management system created multiple points of vulnerability in SISAMEX’s operations. With heavy dependence on manual tasks, the process was particularly susceptible to human errors, especially during data entry and calculations. The absence of defined filters made it challenging to efficiently classify and segment information, leading to inconsistencies in data organization. Adding to these challenges, the lack of standardization meant that each planner developed their own method for handling inventory data, resulting in varying approaches and inconsistent results across the team. The situation was further complicated by the cumbersome process of manually tracking and consolidating inventory reports from enterprise resource planning (ERP), which consumed significant time and created opportunities for discrepancies in the final reports. These combined factors made it difficult to maintain data accuracy and reliability, ultimately affecting the quality of decision-making processes.</li> 
 <li><strong>System and visibility limitations –</strong> A challenge SISAMEX faced was the absence of real-time visibility into their aged inventory status. The company’s data infrastructure was fragmented across multiple systems, including their ERP system, electronic data interchange (EDI) documents, and various legacy systems, creating information silos that hindered effective decision-making. Without automated follow-up capabilities, the team struggled to maintain consistent monitoring of inventory levels and movement. This fragmentation particularly impacted collaboration with external stakeholders, because the limited ability to share timely information with suppliers and purchasing teams created bottlenecks in the supply chain. Furthermore, the lack of traceability and transparency in the process made it difficult to track changes, identify responsibilities, and maintain accountability throughout the inventory management cycle. These systemic limitations not only affected operational efficiency but also impacted the company’s ability to respond quickly to industry demands and maintain optimal inventory levels</li> 
</ul> 
<p>“Before automation, our teams were heavily dependent on manual tasks, leading to significant time investments and potential errors in our inventory management process,” explains Leslie Paz from the Supply Chain team. The situation demanded a more efficient, automated solution to improve response times and data accuracy.</p> 
<h2><strong>The solution: Digital transformation with Amazon Quick Sight</strong></h2> 
<p>Working with <a href="https://www.xaldigital.com/" rel="noopener noreferrer" target="_blank">XalDigital</a>, an AWS Partner, SISAMEX implemented a comprehensive analytics solution using on Quick Sight that achieved a remarkable 99% reduction in processing time – from 3 hours to only 1 minute. This transformation has revolutionized their inventory management capabilities through complete automation, real-time data visibility, and enhanced supplier collaboration.</p> 
<p>By centralizing data and automating manual processes, the solution minimizes human errors and standardized operations across teams. The system´s implementation marked a pivotal shift from time-consuming manual analysis to strategic decision-making, enabling SISAMEX’s team members to focus on value-adding activities rather than data processing. With 18 active users across the organization now using this solution, SISAMEX has established a new standard for data-driven inventory management in the automotive manufacturing sector.</p> 
<h2><strong>The implementation included a Pay Over Consumption</strong></h2> 
<p>Dashboard in Quick Sight provides a comprehensive view of inventory aging with color-coded thresholds, trend analysis, and detailed supplier metrics. The following screenshot shows this dashboard.</p> 
<p><img alt="" class="alignnone size-full wp-image-6462" height="699" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/07/BI-7345-image-1.png" width="1086" /></p> 
<p>With Quick Sight direct query capabilities, SISAMEX gained access to inventory data in near real time (less than 1 second refresh). Planners can visualize aging inventory patterns and make immediate decisions on stock management. This real-time visibility replaced the previous 3-hour delay in data access and analysis. Using Quick Sight robust filtering and drill-down features, teams can now perform detailed supplier-level analysis, enabling them to identify trends, monitor performance, and track inventory levels by individual supplier. This granular visibility has improved supplier relationship management and reduced response times from 3 hours to mere minutes. Implementation of Quick Sight conditional formatting and custom alerts created an intuitive traffic light system that automatically flags important inventory levels. This visual management system helps teams quickly identify:</p> 
<ul> 
 <li><strong>Green</strong> – Optimal inventory levels</li> 
 <li><strong>Yellow</strong> – Warning thresholds requiring attention</li> 
 <li><strong>Red</strong> – Urgent levels needing immediate action</li> 
</ul> 
<p>The following screenshot shows the Strategic Dashboard, which provides a granular view of inventory aging by supplier, with detailed monetary values, quantity tracking, and a real-time pie chart showing the distribution of aging inventory across different time periods.</p> 
<p><img alt="" class="alignnone size-full wp-image-6463" height="1134" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/07/BI-7345-image-2.png" width="1428" /></p> 
<p>Through Quick Sight data preparation capabilities and integration with <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3</a>), the solution automatically consolidates data from multiple sources (such as ERP, EDI and legacy systems). This automation removes the previous 3-hour manual consolidation process by implementing automatic Super-fast, <a href="https://docs.aws.amazon.com/quicksuite/latest/userguide/refreshing-imported-data.html" rel="noopener noreferrer" target="_blank">Parallel, In-memory Calculation Engine (SPICE)</a> incremental refreshes every 15 minutes, keeping data current and reducing computational overhead by only updating changed records.</p> 
<p>Using Quick Sight dynamic dashboard features, SISAMEX developed customized key performance indicators (KPIs) that track:</p> 
<ul> 
 <li>Inventory aging by category</li> 
 <li>Supplier performance metrics</li> 
 <li>Stock turnover rates</li> 
 <li>Cost implications of inventory decisions</li> 
</ul> 
<p>These metrics are now automatically updated and accessible to 18 active users across the organization.</p> 
<h2><strong>Technical architecture</strong></h2> 
<p>SISAMEX modernized our data architecture by integrating Quick Sight with our existing ERP system database, centralizing data storage in S3 data lakes, implementing automated extract, transform, and load (ETL) pipelines with AWS Glue, and deploying mobile-optimized dashboards, transforming manual processes into a scalable, cloud-based analytics solution. The solution architecture showcases the seamless integration between source systems, Quick Sight, and other AWS services, enabling automated data flow from git repository and databases to end users. It offers:</p> 
<p><img alt="" class="alignnone size-full wp-image-6464" height="945" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/07/BI-7345-image-3.png" width="1276" /></p> 
<ul> 
 <li><strong>Cloud-based data storage in Amazon S3</strong> – Managing multiple Excel files across different locations created significant challenges in maintaining accurate inventory data. The team constantly struggled with version control and data accessibility issues. Moving to a centralized data lake in Amazon S3 helped create a scalable and more secure foundation for Quick Sight analytics. This centralization established a single source of truth, reducing version control issues and enabling real-time updates for 18 users across the organization.</li> 
 <li><strong>Automated data processing workflows –</strong> The lack of standardization meant each planner developed their own process for data analysis, resulting in inconsistent results and significant time investment in manual calculations. Through automated ETL pipelines, consolidating data from multiple data sources in S3, and using custom SQL in Quick Sight data preparation through Athena to gather the data, SISAMEX standardized their entire data processing workflow. This automation dramatically reduced analysis time, removed manual errors, and enabled consistent reporting methodologies across teams.</li> 
 <li><strong>Mobile-friendly dashboard access – </strong>Team members often found themselves unable to access vital inventory data outside the office, leading to delayed decision-making and slow response times to suppliers. By using Quick Sight responsive design capabilities, SISAMEX created mobile-optimized dashboards that transformed how teams accessed information. Now, team members can view real-time inventory data and make crucial decisions from many devices, significantly improving response times to suppliers and internal stakeholders.</li> 
</ul> 
<blockquote>
 <p><em>“Thanks to the new Quick Sight service, we’ve optimized decision-making times for our Materials Planning team significantly,” shares Alondra Elizondo from the Supply Chain team. </em></p>
</blockquote> 
<p>This modern architecture, shown in the diagram, has fundamentally transformed SISAMEX’s data management capabilities, creating a robust foundation for future scaling and analytics innovations. The solution addressed immediate operational challenges and positioned SISAMEX for continued growth and efficiency improvements.</p> 
<h2><strong>Implementation journey</strong></h2> 
<p>The transformation was carried out in four strategic phases:</p> 
<p><strong>Phase 1: Data integration</strong></p> 
<ul> 
 <li>Created centralized data lake in Amazon S3</li> 
 <li>Developed automated data collection queries</li> 
 <li>Established data quality controls</li> 
</ul> 
<p><strong>Phase 2: Dashboard development</strong></p> 
<ul> 
 <li>Built intuitive visualization interfaces</li> 
 <li>Implemented dynamic filtering capabilities</li> 
 <li>Created automated reporting systems</li> 
</ul> 
<p><strong>Phase 3: User adoption</strong></p> 
<ul> 
 <li>Trained more than 18 active users across three departments</li> 
 <li>Established best practices</li> 
 <li>Developed standard operating procedures</li> 
</ul> 
<p><strong>Phase 4: Continuous improvement</strong></p> 
<ul> 
 <li>Regular feedback collection</li> 
 <li>Feature enhancement based on user needs</li> 
 <li>Integration of AI capabilities</li> 
</ul> 
<h2><strong>Measurable results</strong></h2> 
<p>The implementation of Quick Sight has delivered transformative results across SISAMEX’s operations. Most notably, the solution dramatically reduced analysis time from 3 hours to 1 minute, freeing up valuable resources for strategic activities. The automation of data consolidation and report generation has minimized human errors in calculations and resulted in standarized inventory classification processes across the organization. This improvement in data quality and reliability has fundamentally changed how teams operate, enabling real-time inventory visibility and reducing supplier response times from hours to minutes. The enhanced decision-making capabilities have fostered stronger cross-departmental collaboration, with 18 active users now accessing consistent, accurate data through a single system. The standardization of processes has improved operational efficiency and also created a more transparent and agile environment where teams can focus on strategic analysis rather than manual data processing. “My greatest satisfaction is seeing how this project helps the Planning team reduce operational time and become more strategic. We managed to implement these dashboards in record time to meet business needs with new challenges,” notes Leslie Paz from the Supply Chain team</p> 
<p>SISAMEX plans to enhance their analytics capabilities by using <a href="https://aws.amazon.com/sagemaker/ai/" rel="noopener noreferrer" target="_blank">Amazon SageMaker AI</a> for machine learning (ML), enabling predictive maintenance scheduling and demand forecasting across their manufacturing operations.</p> 
<p><strong>Expanding analytics with Amazon Quick Suite agentic capabilities</strong></p> 
<p>The team is also expanding their analytics capabilities deeper into their supply chain operations, moving beyond inventory management to encompass end-to-end supply chain visibility. By using Quick Suite natural language querying capabilities, SISAMEX aims to democratize data access across their organization, enabling team members at different levels to interact with data through simple, conversational queries; Users can take advantage of the solution´s advanced querying features to access data insights through intuitive interfaces designed for users at varying technical levels; Team members can use the natural language capabilities in Quick Suite to retrieve data insights through conversational interfaces.</p> 
<p>The following screenshot shows how agentic AI capabilities in Quick Suite provide AI-powered natural language insights, automatically generating detailed supplier analysis and actionable recommendations from complex inventory data.</p> 
<p><img alt="" class="wp-image-6468 size-full aligncenter" height="1280" src="https://d2908q01vomqb2.cloudfront.net/d02560dd9d7db4467627745bd6701e809ffca6e3/2026/01/07/BI-7345-image-4-new.png" width="500" /></p> 
<p>SISAMEX envisions deploying custom automated workflows tailored to specific manufacturing and supply chain scenarios, providing specialized support for different operational areas from production planning to quality control. The company plans to implement domain-specific automation tools configured with relevant data sources and connected systems to orchestrate complex workflows. Future enhancements will include specialized query interfaces for different operational areas, each optimized for specific manufacturing and supply chain use cases. SISAMEX plans to leverage automation capabilities in Quick Suite to transform their supply chain processes through:</p> 
<ul> 
 <li>Implementation of <a href="https://aws.amazon.com/quicksuite/flows/" rel="noopener noreferrer" target="_blank">Amazon Quick Flows</a> and <a href="https://aws.amazon.com/quicksuite/automate/" rel="noopener noreferrer" target="_blank">Amazon Quick Automate</a> to create intelligent, self-optimizing supply chain processes</li> 
 <li>Creation of automated workflows that business users can build and customize without technical skills using natural language or chat agent interactions</li> 
 <li>Development of a comprehensive daily monitoring system that tracks important business metrics and responds to significant changes in real-time</li> 
 <li>Automated tracking of specific data changes in dashboards, including customer categories and product lines, with automatic calculation of month-over-month variations and threshold monitoring</li> 
 <li>Transformation of routine monitoring into proactive business intelligence systems that can adapt to industry changes while keeping decision-makers informed</li> 
</ul> 
<p>This advanced automation strategy means that SISAMEX can build intelligent workflows that combine process automation with sophisticated monitoring capabilities, creating a more responsive and efficient supply chain operation.</p> 
<h2><strong>Conclusion</strong></h2> 
<p>SISAMEX has transformed their inventory management processes. This digital transformation has delivered multiple strategic benefits: minimization of human errors through automation, standardization of processes across teams, and real-time visibility for better decision-making. The implementation has improved operational efficiency and enhanced collaboration with suppliers and internal stakeholders. With more than 18 active users now using the system daily, SISAMEX has created a data-driven culture that enables proactive inventory management rather than reactive problem-solving. The success of this project demonstrates how cloud-based analytics can transform traditional manufacturing operations into agile, data-driven organizations.</p> 
<hr /> 
<h3><strong>About the authors</strong></h3> 
<p><strong>Francisco Gutiérrez</strong> is the IT Manager at SISAMEX, where he leads technology infrastructure and digital transformation initiatives. With extensive experience in automotive industry IT operations, he drives innovation in cloud adoption and business intelligence solutions to optimize supply chain and operational efficiency.</p> 
<p><strong>Belinda Quintanilla</strong> serves as Business Intelligence Leader at SISAMEX, specializing in data analytics and insights that drive strategic decision-making across the organization. She leverages advanced BI tools and cloud technologies to transform raw data into actionable intelligence, enabling SISAMEX to help operational performance and customer service.</p> 
<p><strong>José Baños</strong> serves as Go-To-Market (GTM) Lead for Amazon Quick Suite across the Americas at Amazon Web Services (AWS), where he drives strategic adoption of AI-powered business intelligence and workplace productivity solutions. He has deep expertise in enterprise software, data analytics, and cloud transformation.</p> 
<p><strong>Diego Acevedo</strong> is an Enterprise Account Manager at Amazon Web Services (AWS). He helps organizations accelerate their cloud journey while driving business impact through data-driven strategies and customer-centric innovation.</p> 
<p><strong>Daniel Bravo</strong> has been a Solutions Architect at AWS since 2020, with experience serving clients in Hospitality &amp; Travel, Manufacturing, and Transportation &amp; Logistics industries. He specializes in Data Analytics and Data Engineering, focusing on data integration.</p>
