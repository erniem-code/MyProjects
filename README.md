# FPC | Canadian Invoice Advanced PDF/HTML Template<img width="468" height="62" alt="image" src="https://github.com/user-attachments/assets/d838b532-cf65-4119-b1ba-765a0ae26707" />

Overview
This document explains the XML-based NetSuite Advanced PDF/HTML Template for the FPC | Canadian Invoice. The FreeMarker template generates a customer-facing invoice PDF with a header, billing and shipping addresses, line items, and a summary table. Conditional GST/HST and PST tax rows appear only for Canadian shipping destinations.

Template type: NetSuite Advanced PDF/HTML Template (FreeMarker + XML/HTML)
Record type: Invoice (Transaction)
Output format: PDF (Letter size)
Target audience: Canadian customers (conditional tax display)

File Structure
The template is organized into the following logical sections:

<img width="468" height="310" alt="image" src="https://github.com/user-attachments/assets/98ab9e05-d658-4a96-b8fa-417be02487f4" />
Section	Element	Purpose
Prolog	<?xml ...?>, <!DOCTYPE pdf ...>	XML declaration and NetSuite PDF DTD reference
Head	<head>	Font links, macros (header/footer), CSS styles
Fonts	<link name="NotoSans" ...>	Multi-language font declarations (Latin + CJK + Thai)
Macros	<macrolist>	Reusable header (nlheader) and footer (nlfooter) blocks
Styles	<style type="text/css">	CSS for tables, totals, addresses, typography
Body	<body>	Main invoice content: addresses, metadata, items, summary
<img width="468" height="223" alt="image" src="https://github.com/user-attachments/assets/5b0e61d9-4631-44c2-8114-4ea196f9142b" />

Header Section

Font Declarations
<link name="NotoSans" ... src="${nsfont.NotoSans_Regular}" ... />

•	Primary font: NotoSans (regular, bold, italic, bold-italic)
•	Conditional CJK/Thai fonts loaded based on .locale:
◦	zh_CN → NotoSansCJKsc
◦	zh_TW → NotoSansCJKtc
◦	ja_JP → NotoSansCJKjp
◦	ko_KR → NotoSansCJKkr
◦	th_TH → NotoSansThai
•	Fallback: NotoSans only
<img width="468" height="279" alt="image" src="https://github.com/user-attachments/assets/87bbd747-5a90-413c-a694-48597ed57d09" />

Header Macro (nlheader)

Renders at the top of every page:
Element	Field	Description
Logo	${companyInformation.logoUrl}	Company logo from File Cabinet
Company address	${companyInformation.addressText}	Full company address
Document title	${record@title}	e.g., "Invoice"
Document number	#${record.tranid}	Transaction ID with # prefix
Date	${record.trandate}	Transaction date
Memo	${record.memo}	Transaction memo

Footer Macro (nlfooter)

Renders at the bottom of every page:
Element	Field	Description
Barcode	<barcode codetype="code128" value="${record.tranid}"/>	Code128 barcode of transaction ID
Page number	<pagenumber/> of <totalpages/>	e.g., "Page 1 of 3"

Style Section

Key CSS classes and their purposes:
Selector	Purpose
table	Base font size 9pt, fixed layout
th	Bold, 8pt, gray background (#e3e3e3), padded
td	4px/6px padding
table.header td	Header cells — no padding, 10pt
table.footer td	Footer cells — no padding, 8pt
table.total	page-break-inside: avoid — keeps summary on one page
tr.totalrow	Gray background, 200% line height (Total row)
span.title	28pt (document title)
span.number	16pt (transaction number)
hr	1px gray divider

Body Section — Content Blocks
Bill-To / Ship-To Addresses
<table style="width: 100%; margin-top: 10px;">

Column	Field
Left (colspan 3)	${record.billaddress} — billing address
Right (colspan 3)	${record.shipaddress} — shipping address

Labels rendered via ${record.billaddress@label} and ${record.shipaddress@label}.

Transaction Metadata
A 5-column table:
Column	Field
Terms	${record.terms}
Due Date	${record.duedate}
PO/Ref #	${record.otherrefnum}
Sales Rep	${record.salesrep}
Ship Method	${record.shipmethod}

Line Items Table

Rendered only when <#if record.item?has_content>:
Column	Field	Alignment
Quantity	${item.quantity}	Center
Item #	${item.item}	Left
Description	${item.description}	Left
Rate	${item.rate}	Right
Amount	${item.amount}	Right

Iterates via <#list record.item as item> with <#if item_index==0> for the header row.

Summary / Totals Table
Structure: 6-column table (spacer colspan=4, label, amount)

Row	Field	Conditional
Subtotal	${record.subtotal}	Always
GST/HST	${record.taxtotal} with rate	Gated by ship-to country = Canada
PST	${record.tax2total}	Gated by ship-to country = Canada AND tax2total ≠ 0
Shipping & Handling	${record.shippingcost}	Always
Total	${record.total}	Always (highlighted totalrow)
Remittance	${record.custbody_remittance_address}	Always

Conditional Tax Logic
The Gate Condition
<#assign shipToCountry = (record.shipcountrycode!"")?upper_case?trim>
<#if shipToCountry == "CA" || shipToCountry == "CANADA">
    <!-- GST/HST row -->
    <#if record.tax2total?has_content && record.tax2total != 0>
        <!-- PST row -->
    </#if>
</#if>

Normalization Pipeline

Step	Operation	Purpose
1	record.shipcountrycode!""	Fallback to empty string if null
2	?upper_case	Handle ca, Ca, Canada, CANADA
3	?trim	Strip leading/trailing whitespace

Match Logic
Normalized value	Match?
CA	✅ Yes
CANADA	✅ Yes
US, USA, UNITED STATES	❌ No
"" (empty)	❌ No
CAN (ISO alpha-3)	❌ No (known limitation)

PST Additional Gate
PST row requires:
•	record.tax2total?has_content (field exists and is non-empty)
•	record.tax2total != 0 (non-zero amount)

Behavior Matrix
Ship-To Country	GST/HST Row	PST Row
Canada, tax2total ≠ 0	✅ Shows	✅ Shows
Canada, tax2total = 0	✅ Shows	❌ Hidden
USA / other	❌ Hidden	❌ Hidden
Empty / null	❌ Hidden	❌ Hidden

FreeMarker Field Reference
Record Fields Used
Field	Type	Source	Usage
${record@title}	String	Record type	Document title
${record.tranid}	String	Transaction	Document number, barcode
${record.trandate}	Date	Transaction	Invoice date
${record.memo}	String	Transaction	Memo
${record.billaddress}	Address	Transaction	Bill-to
${record.shipaddress}	Address	Transaction	Ship-to
${record.shipcountrycode}	String	Ship-to address	Tax gate
${record.terms}	String	Transaction	Payment terms
${record.duedate}	Date	Transaction	Due date
${record.otherrefnum}	String	Transaction	PO/Reference
${record.salesrep}	String	Transaction	Sales rep
${record.shipmethod}	String	Transaction	Ship method
${record.subtotal}	Number	Calculated	Subtotal
${record.taxtotal}	Number	Calculated	GST/HST amount
${record.taxrate}	Number	Tax code	GST/HST rate
${record.tax2total}	Number	Calculated	PST amount
${record.shippingcost}	Number	Transaction	Shipping
${record.total}	Number	Calculated	Grand total
${record.custbody_remittance_address}	String	Custom body field	Remittance address

Company Fields

Field	Usage
${companyInformation.logoUrl}	Logo image
${companyInformation.addressText}	Company address

Dependencies & Prerequisites

Dependency	Type	Notes
NotoSans fonts	NetSuite system fonts	${nsfont.NotoSans_*} — must be available in account
CJK/Thai fonts	System fonts	Only required if multi-locale support is needed
tax2total field	Standard NetSuite field	Must be populated by a secondary tax code (e.g., PST) on line items
custbody_remittance_address	Custom body field	Must exist on the Invoice record type
Ship-To address	Transaction data	Must be populated for the tax gate to evaluate correctly

Known Limitations
Limitation	Impact	Mitigation
ISO alpha-3 not matched	CAN → tax rows hidden	Add || shipToCountry == "CAN" to condition if instance returns alpha-3
Tab/newline whitespace	?trim may not strip all whitespace types	Test; use ?replace if needed
String-typed tax2total	"0.00" != 0 may behave unexpectedly	Use record.tax2total?number != 0 with ?has_content guard
Negative PST (credits)	Negative PST displays	Add record.tax2total > 0 if credits should hide PST
GST/HST shows $0.00	GST/HST row has no zero-check	Wrap in <#if record.taxtotal != 0> if zero should hide

Change Log
Version	Change	Date
v1.0	Base template	—
v1.1	Added GST/HST + PST rows to summary table	October 21, 2026
v1.2	PST conditional on tax2total != 0	October 22, 2026
v1.3	PST conditional on ship-to = Canada AND ship-from = Canada	October 22, 2026
v1.4	Removed ship-from check (PST gated by ship-to only)	October 22, 2026
v1.5	Gated both GST/HST and PST by ship-to country	October 22, 2026
v1.6	Robust country normalization (?upper_case?trim, match CA OR CANADA)	October 23, 2026

Testing
Refer to the accompanying Test Cases document (35 test cases) covering:
•	Country value formats (ISO, full name, case variants, whitespace)
•	tax2total values (zero, non-zero, negative, empty, null)
•	Non-Canadian ship-to scenarios
•	Multi-page, no-line-items, locale, and regression tests

Test execution: Preview via Advanced PDF/HTML template editor or print-to-PDF from test transactions; visually verify row presence and values.

Deployment Procedure
1.	Backup current template XML (download from NetSuite).
2.	Edit template: Customization > Forms > Advanced PDF/HTML Templates > FPC | Canadian Invoice.
3.	Paste updated XML into the template source.
4.	Preview with a Canadian transaction (ship-to = Canada, PST applied) — confirm GST/HST + PST rows.
5.	Preview with a US transaction — confirm no tax rows.
6.	Save the template.
7.	Assign to the appropriate transaction form(s) if not already assigned.
8.	Monitor printed invoices for the first week post-deployment.

Support & Maintenance

Item	Owner	Frequency
Template review	NetSuite Admin	Quarterly
Test suite execution	QA	After any change
Country code data validation	Data team	On import of new customers
Font availability check	IT	On NetSuite release upgrades

Escalation: Contact NetSuite Administrator for template access issues; Data team for country-code anomalies.

14. Glossary

Term	Definition
GST/HST	Goods and Services Tax / Harmonized Sales Tax (Canadian federal tax)
PST	Provincial Sales Tax (Canadian provincial tax)
Ship-To	Customer delivery address on the transaction
Ship-From	Company origin address (no longer used in v1.5+)
FreeMarker	Templating language used by NetSuite Advanced PDF/HTML
Advanced PDF/HTML	NetSuite's PDF rendering engine for templates
tax2total	NetSuite field for secondary tax total (PST)
taxtotal	NetSuite field for primary tax total (GST/HST)

<img width="468" height="647" alt="image" src="https://github.com/user-attachments/assets/ecc6b2f4-32f0-4a8d-862a-b2eeb65c190d" />
