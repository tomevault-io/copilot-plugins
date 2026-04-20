## marketinout

> You can select a sector or industry as one of the stock screening criteria in the Stock Screener or the Formula Stock Screener. The list of all supported sectors and industries is given below. In the Stock Screener tool, a sector or industry is selected by name from the Stock Universe criteria category. In the Formula Screener, a sector or industry condition can be added using the corresponding sector or industry functions, the examples of which are given below.


You can select a sector or industry as one of the stock screening criteria in the Stock Screener or the Formula Stock Screener. The list of all supported sectors and industries is given below. In the Stock Screener tool, a sector or industry is selected by name from the Stock Universe criteria category. In the Formula Screener, a sector or industry condition can be added using the corresponding sector or industry functions, the examples of which are given below.


# List of Sectors

Sector - Formula Token
Energy - energy
Basic Materials - basic_materials
Industrials - industrials
Consumer Defensive - consumer_defensive
Consumer Cyclical - consumer_cyclical
Healthcare - healthcare
Financial Services - financial_services
Technology - technology
Communication Services - communication_services
Utilities - utilities
Real Estate - real_estate

Formula usage example: sector(consumer_cyclical,healthcare,technology)

The formula expression for screening Exchange Traded Funds (ETFs) remains operational without the need for change: sector(etf). However, it is worth noting that the formula expression sector(etf) will be equivalent to the expression type(fund) using the recently added function type, and the expression not sector(etf) will be equivalent to type(stock).

# List of Industries

Code - Sector - Industry
110 - Basic Materials - Chemicals
112 - Basic Materials - Agricultural Inputs
113 - Basic Materials - Specialty Chemicals
130 - Basic Materials - Steel
131 - Basic Materials - Copper
132 - Basic Materials - Aluminum
133 - Basic Materials - Other Industrial Metals & Mining
134 - Basic Materials - Gold
135 - Basic Materials - Silver
136 - Basic Materials - Nonmetallic Mineral Mining
137 - Basic Materials - Other Precious Metals & Mining
138 - Basic Materials - Coking Coal
324 - Basic Materials - Paper & Paper Products
632 - Basic Materials - Lumber & Wood Production
634 - Basic Materials - Building Materials
734 - Consumer Defensive - Grocery Stores
732 - Consumer Defensive - Discount Stores
766 - Consumer Defensive - Education & Training Services
757 - Consumer Defensive - Food Distribution
323 - Consumer Defensive - Household & Personal Products
340 - Consumer Defensive - Packaged Foods
341 - Consumer Defensive - Farm Products
345 - Consumer Defensive - Confectioners
346 - Consumer Defensive - Beverages - Brewers
347 - Consumer Defensive - Beverages - Wineries & Distilleries
348 - Consumer Defensive - Beverages - Non-Alcoholic
350 - Consumer Defensive - Tobacco
351 - Consumer Cyclical - Travel Services
352 - Consumer Cyclical - Internet Retail
325 - Consumer Cyclical - Packaging & Containers
330 - Consumer Cyclical - Auto Manufacturers
332 - Consumer Cyclical - Recreational Vehicles
333 - Consumer Cyclical - Auto Parts
311 - Consumer Cyclical - Furnishings Fixtures & Appliances
320 - Consumer Cyclical - Apparel Manufacturing
321 - Consumer Cyclical - Footwear & Accessories
710 - Consumer Cyclical - Lodging
711 - Consumer Cyclical - Resorts & Casinos
712 - Consumer Cyclical - Restaurants
714 - Consumer Cyclical - Gambling
715 - Consumer Cyclical - Leisure
762 - Consumer Cyclical - Personal Services
736 - Consumer Cyclical - Home Improvement Retail
739 - Consumer Cyclical - Specialty Retail
742 - Consumer Cyclical - Luxury Goods
744 - Consumer Cyclical - Auto & Truck Dealerships
628 - Consumer Cyclical - Textile Manufacturing
630 - Consumer Cyclical - Residential Construction
730 - Consumer Cyclical - Apparel Retail
731 - Consumer Cyclical - Department Stores
827 - Financial Services - Financial Data & Stock Exchanges
928 - Financial Services - Banks - Regional
210 - Financial Services - Shell Companies
447 - Financial Services - Mortgage Finance
450 - Financial Services - Financial Conglomerates
451 - Financial Services - Insurance - Diversified
452 - Financial Services - Insurance - Reinsurance
417 - Financial Services - Banks - Diversified
420 - Financial Services - Capital Markets
422 - Financial Services - Asset Management
424 - Financial Services - Credit Services
430 - Financial Services - Insurance - Life
431 - Financial Services - Insurance - Specialty
432 - Financial Services - Insurance - Property & Casualty
434 - Financial Services - Insurance Brokers
512 - Healthcare - Drug Manufacturers - Specialty & Generic
515 - Healthcare - Biotechnology
520 - Healthcare - Medical Instruments & Supplies
521 - Healthcare - Medical Devices
522 - Healthcare - Healthcare Plans
524 - Healthcare - Medical Care Facilities
525 - Healthcare - Diagnostics & Research
529 - Healthcare - Drug Manufacturers - General
733 - Healthcare - Pharmaceutical Retailers
754 - Healthcare - Medical Distribution
825 - Healthcare - Health Information Services
764 - Industrials - Staffing & Employment Services
765 - Industrials - Security & Protection Services
760 - Industrials - Specialty Business Services
761 - Industrials - Rental & Leasing Services
769 - Industrials - Consulting Services
770 - Industrials - Airlines
772 - Industrials - Airports & Air Services
773 - Industrials - Integrated Freight & Logistics
774 - Industrials - Trucking
775 - Industrials - Marine Shipping
776 - Industrials - Railroads
751 - Industrials - Building Products & Equipment
752 - Industrials - Industrial Distribution
635 - Industrials - Infrastructure Operations
636 - Industrials - Engineering & Construction
637 - Industrials - Waste Management
638 - Industrials - Conglomerates
836 - Industrials - Electrical Equipment & Parts
611 - Industrials - Aerospace & Defense
620 - Industrials - Farm & Heavy Construction Machinery
622 - Industrials - Specialty Industrial Machinery
623 - Industrials - Pollution & Treatment Controls
624 - Industrials - Tools & Accessories
626 - Industrials - Metal Fabrication
313 - Industrials - Business Equipment & Supplies
721 - Communication Services - Advertising Agencies
722 - Communication Services - Entertainment
723 - Communication Services - Broadcasting
727 - Communication Services - Publishing
840 - Communication Services - Telecom Services
851 - Communication Services - Internet Content & Information
820 - Communication Services - Electronic Gaming & Multimedia
824 - Technology - Information Technology Services
826 - Technology - Software - Infrastructure
815 - Technology - Computer Hardware
755 - Technology - Electronics & Computer Distribution
841 - Technology - Communication Equipment
837 - Technology - Scientific & Technical Instruments
832 - Technology - Solar
833 - Technology - Semiconductors
834 - Technology - Semiconductor Equipment & Materials
835 - Technology - Electronic Components
925 - Technology - Software - Application
314 - Technology - Consumer Electronics
911 - Utilities - Utilities - Regulated Electric
912 - Utilities - Utilities - Regulated Gas
913 - Utilities - Utilities - Diversified
914 - Utilities - Utilities - Regulated Water
918 - Utilities - Utilities - Independent Power Producers
919 - Utilities - Utilities - Renewable
920 - Energy - Thermal Coal
150 - Energy - Oil & Gas Drilling
151 - Energy - Uranium
120 - Energy - Oil & Gas Integrated
121 - Energy - Oil & Gas E&P
122 - Energy - Oil & Gas Refining & Marketing
124 - Energy - Oil & Gas Equipment & Services
125 - Energy - Oil & Gas Midstream
440 - Real Estate - REIT - Diversified
441 - Real Estate - REIT - Office
442 - Real Estate - REIT - Healthcare Facilities
443 - Real Estate - REIT - Hotel & Motel
444 - Real Estate - REIT - Industrial
445 - Real Estate - REIT - Residential
446 - Real Estate - REIT - Retail
471 - Real Estate - Real Estate - Diversified
472 - Real Estate - REIT - Mortgage
473 - Real Estate - REIT - Specialty
448 - Real Estate - Real Estate Services
449 - Real Estate - Real Estate - Development

Formula usage example: industry(131,134,135)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MakiBM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
