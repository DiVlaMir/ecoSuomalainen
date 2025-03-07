# Global Carbon Balance: By fact, There Are Only a Few Violators

## TLDR: 

We share one planet with a common atmosphere, so anyone can harm it, but no one can escape the consequences. For a fair global carbon balance, each country must strive for its own balance, either by having enough forests to offset its emissions or by reducing emissions to a level that its forests can handle (not more than 5 ton/ha of forest per year). 
The good news is that the results show that the vast majority of countries now meet the criteria. "Good, one less thing to worry about".
Result graph - by country CO₂ emissions by forest area (kg/ha per year), bars are built with logarithmic scale.

## The Importance of Global Carbon Balance

The global carbon balance refers to the equilibrium between carbon dioxide (CO₂) emissions into the atmosphere and its absorption by ecosystems, oceans, and carbon capture technologies. Achieving this balance is crucial for stabilizing the climate and preventing the catastrophic consequences of global warming.

### Why it matters to keep the levels lower?

1. Slowing down climate change: Excess CO₂ enhances the greenhouse effect, leading to rising temperatures, changing weather patterns, and intensifying extreme events (hurricanes, droughts, floods). Balancing carbon emissions can help limit global warming to 1.5–2°C.
2. Preserving ecosystems: Climate change is destroying natural ecosystems: coral reefs are dying due to warming oceans, forests are suffering from fires, and the disappearance of Arctic ice threatens polar fauna.
3. Ensuring food security: Extreme heat waves, droughts, and floods reduce crop yields and the quality of agricultural products, which can lead to food crises and higher food prices.
4. Protecting human health: Air pollution associated with CO₂ and other greenhouse gas emissions causes respiratory and cardiovascular diseases. A sustainable carbon balance helps improve air quality.
5. Economic stability: Climate change harms the economy through the destruction of infrastructure, reduced agricultural productivity, and increased costs of eliminating the consequences of natural disasters. Maintaining balance reduces these risks.
6. Reducing the risk of social and geopolitical conflicts: Climate change exacerbates water and food shortages, forces people to migrate, which can lead to an escalation of international conflicts and political instability.

### How to Achieve Carbon Balance

- Restoration and preservation of forests, wetlands, and marine ecosystems
- Managing forests in order to make those capture more carbon during active restoration phase
- Developing renewable energy sources (solar, wind, hydropower)
- Increasing energy efficiency in industry and everyday life (electric vehicles, public transport, additional insulation, heat pumps for heating etc).

### The Role of Forests in Carbon Absorption

Forests play a crucial role in absorbing CO₂ from the air. On average, forests absorb about 5 tonnes of CO₂ per hectare per year (close to 3 ton/ha for natural mature forests and up to 15 ton/ha for managed harvested forests). The adsorption level also depends much on the tree species and weather conditions, so we'll use 5 ton/ha as a target safe figure, 15 ton/ha as a warning level and 30+ ton/ha as definitely critical issue level, which can't be kept as-is. This will give us a rough estimate of who is doing well and who needs to improve.
To achieve a global carbon balance, each country should aim to have enough forests to offset its emissions or reduce emissions to a level that its forests can handle. This can be calculated using the following formula:

**E(c) ≤ F(c) × T**

where:

E(c) is the annual CO₂ emissions by country,
F(c) is the forest area by country,
T is the target CO₂ adsorption by forests (5 ton/ha).

## Checking the current levels - let’s take the real world data and build a graph

Let's build a graph of CO2 emissions per hectare of forest per year for all the countries taking into account the most recent data on forest area and CO2 emissions per country.
We can take the most latest CO2 emission data by country from OWID data-file like this:

```py
import json

emissionsData = {}
with open('owid-co2-data.json') as json_file:
    data = json.load(json_file)
    for country, country_data in data.items():
        for yearly_data in country_data['data']:
            if yearly_data['year'] >= 2020 and 'co2' in yearly_data:
                emissionsData[country] = yearly_data['co2']
In other sources we sometimes have country names written differently, let's keep them all inline using this function:
def cleanCountryName(country):
    if country == 'Cabo Verde':
        return 'Cape Verde'
    if country == 'Korea, North':
        return 'North Korea'
    if country == 'Korea, South':
        return 'South Korea'
    if country == 'Turkey (Turkiye)':
        return 'Turkey'
    if country == 'Czech Republic':
        return 'Czechia'
    if country == 'Congo, Democratic Republic of the':
        return 'Democratic Republic of Congo'
    if country == 'Congo, Republic of the':
        return 'Congo'

    if country == 'Czech Republic':
        return 'Czechia'
    if country == 'Gambia, The':
        return 'Gambia'
    if country == 'Republic Of The Congo':
        return 'Democratic Republic of Congo'
    if country == 'Congo, Republic of the':
        return 'Congo'
    if country == 'Virgin Islands':
        return 'British Virgin Islands'
    
    return country
```
	
We check the land area by country:

```py
import csv
land_area_by_country = {}

with open('Area.csv', 'r', encoding='utf-8-sig') as csvfile:
    reader = csv.DictReader(csvfile, delimiter=',')
    for row in reader:
        country = cleanCountryName(row['name'])
        land_area_by_country[country] = float(row[' sq km'].replace(',', ''))
```

From there we can have the forest area as we have the latest data about share of the forest area to land area per country (we will have the latest data as records are sorted by year):

```py
import csv
forest_area_by_country = {}

with open('forest-area-as-share-of-land-area.csv', 'r', encoding='utf-8') as csvfile:
    reader = csv.DictReader(csvfile, delimiter=',')
    for row in reader:
        country = cleanCountryName(row['Entity'])
        if int(row['Year']) < 2015 or country not in land_area_by_country:
            continue
        forest_area_by_country[country] = float(row['Forest cover']) * land_area_by_country[country]
```

Get country codes to print flags in the graph:

```py
import csv
country_codes = {}

with open('country_codes.csv', 'r', encoding='utf-8-sig') as csvfile:
    reader = csv.DictReader(csvfile, delimiter=';')
    for row in reader:
        country = cleanCountryName(row['country'])
        country_codes[country] = row['code_2_capital']
Collecting the overall data into a single dataset, CO2 is in Mton, so converting to tonnes, area is in sq.km., converting to ha:
overall_data = {}
for country, co2 in emissions_by_country.items():
    if country in country_codes and country in forest_area_by_country:
        data = {}
        data['country'] = country
        data['co2_kg'] = round(co2 * 10**9, 9)
        data['forest_area_ha'] = 0
        if country in forest_area_by_country:
            data['forest_area_ha'] = forest_area_by_country[country] * 100

        if data['forest_area_ha'] > 0:
            data['co2_per_ha_of_forest'] = round(data['co2_kg'] / data['forest_area_ha'], 2)
            overall_data[country] = data
        else:
            print(f"country {country} has no forest at all - shame on it, skipping")
```

Btw, from the above we can see that Quatar and Nauru destroyed their forests completely so now everything they emit is on others.
Create a pandas dataset from the collected data:

```py
import pandas as pd

df1 = pd.DataFrame.from_dict(overall_data, orient='index')
df = pd.DataFrame({'Name': df1['country'], 'Value': df1['co2_per_ha_of_forest']})
```

Generating the graph:

```py
import numpy as np
import os.path
import matplotlib.offsetbox as box
from scipy import ndimage

def get_flag(name):
    path = f"flags\\32x24_wave\\{country_codes[name].lower()}.png"
    im = plt.imread(path)
    return im

# Reorder the dataframe
df = df.sort_values(by=['Value'])

# initialize the figure
plt.figure(figsize=(30,30), dpi=80)
ax = plt.subplot(111, polar=True)
plt.axis('off')

# Constants = parameters controling the plot layout:
upperLimit = 50000
lowerLimit = 1
labelPadding = 0.4
centerPadding = 5

# Compute max and min in the dataset
max = df['Value'].max()
min = df['Value'].min()

# Let's compute heights: they are a conversion of each item value in those new coordinates
# In our example, 0 in the dataset will be converted to the lowerLimit (10)
# The maximum will be converted to the upperLimit (100)
koeff = (upperLimit - lowerLimit) / (max - min)
heights = np.log(koeff * (df.Value - min) + lowerLimit)

# Compute the width of each bar. In total we have 2*Pi = 360°
width = 2*np.pi / len(df.index)

# Compute the angle each bar is centered on:
indexes = list(range(1, len(df.index)+1))
angles = [element * width for element in indexes]

levels = [
    (0, "darkgreen"),   # 0-5 ton is an avg safe level, even for natural non-managed forests 
    (5000, "orange"),   # 5-15 ton is something doable (with managed forests)
    (15000, "darkred"), # 15-30 ton is not safe, but potentially can be adsorbed using some specific tree species
    (30000, "black")    # 30+ can't be treated as normal - the country should take a note from others to do smth
]
log_levels = [(np.log(koeff * (l[0] - min) + lowerLimit), l[1]) for l in levels]

# depending on the bar heights we color it
colors = []
for h in heights:
    for l in log_levels:
        if h >= l[0]:
            color = l[1]
    colors.append(color)

# Draw bars
bars = ax.bar(
    x=angles, 
    height=heights, 
    width=width, 
    bottom=centerPadding,
    linewidth=2, 
    edgecolor="white",
    # color="#61a4b2",
    color=colors,
)

def offset_image(x, y, name, ax):
    img = get_flag(name)
    #if img:
    im = box.OffsetImage(img, zoom=0.72)
    im.image.axes = ax
    ab = box.AnnotationBbox(im, (x, y),  xybox=(0., 16.), frameon=False,
                        xycoords='data',  boxcoords="offset points", pad=0)
    ax.add_artist(ab)

# Add labels
for bar, angle, height, label, value in zip(bars, angles, heights, df["Name"], df["Value"]):
    # Labels are rotated. Rotation must be specified in degrees :(
    rotation = np.rad2deg(angle)

    # Flip some labels upside down
    alignment = ""
    if angle >= np.pi/2 and angle < 3*np.pi/2:
        alignment = "right"
        rotation = rotation + 180
    else: 
        alignment = "left"

    # Finally add the labels
    ax.text(
        x=angle, 
        y=centerPadding + bar.get_height() + labelPadding, 
        s=f"{label} ({value})", 
        ha=alignment, 
        va='center', 
        rotation=rotation, 
        rotation_mode="anchor")

    img = get_flag(label)
    additional_rotation = 0
    if alignment == "left":
        additional_rotation = 180
    #rotated_img = ndimage.rotate(img, rotation + additional_rotation)
    im = box.OffsetImage(img, zoom=0.3)
    im.image.axes = ax
    ab = box.AnnotationBbox(im, (angle, centerPadding + bar.get_height() + 0.2),  xybox=(0., 0.), frameon=False,
                        xycoords='data',  boxcoords="offset points", pad=0)
    ax.add_artist(ab)
```

## Conclusion

Achieving carbon balance is not just an environmental task but a vital necessity for preserving the planet for future generations. It requires collective effort and responsibility from all countries to ensure that our common atmosphere is protected and that the negative effects are not disproportionately borne by some while others continue to benefit from excessive emissions.
But apparently (surprisingly for me) most of the countries are doing good - we definitely do not have any emergency about CO₂ emission levels. Though forests are still critical and should be kept.
According to the historical data about forest area share one can see that most of the countries have those shares slowly declining which is not good.

## Value list ordered by country name

```text
[Country]                 [CO2 (kg/ha per year)]
Afghanistan                                91.28
Albania                                    62.15
Algeria                                   913.97
Andorra                                   266.05
Angola                                      3.12
Anguilla                                  259.89
Antigua and Barbuda                       788.74
Argentina                                  67.45
Armenia                                   221.72
Aruba                                   20647.14
Australia                                  28.40
Austria                                   147.81
Azerbaijan                                370.56
Bahrain                                535542.78
Bangladesh                                528.05
Barbados                                 1891.65
Belarus                                    63.00
Belgium                                  1199.64
Belize                                      5.19
Benin                                      17.86
Bermuda                                  5225.32
Bhutan                                      6.25
Bolivia                                     4.53
Bosnia and Herzegovina                     92.24
Botswana                                    4.29
Brazil                                      9.61
British Virgin Islands                    508.47
Brunei                                    283.13
Bulgaria                                   90.78
Burkina Faso                                9.93
Burundi                                    23.42
Cambodia                                   24.39
Cameroon                                    4.86
Canada                                     14.22
Cape Verde                                116.92
Central African Republic                    0.13
Chad                                        7.00
Chile                                      41.74
China                                     529.31
Colombia                                   17.33
Comoros                                   113.50
Congo                                       3.49
Cook Islands                               47.51
Costa Rica                                 27.26
Croatia                                    90.37
Cuba                                       66.71
Cyprus                                    415.33
Czechia                                   313.12
Democratic Republic of Congo                0.35
Denmark                                   402.70
Djibouti                                  848.48
Dominica                                   34.02
Dominican Republic                        148.23
Ecuador                                    30.26
Egypt                                   59447.42
El Salvador                               138.81
Equatorial Guinea                          25.08
Eritrea                                     5.05
Estonia                                    39.49
Eswatini                                   21.98
Ethiopia                                    9.12
Faroe Islands                           99916.72
Fiji                                       10.05
Finland                                    12.68
France                                    134.32
French Polynesia                           53.84
Gabon                                       2.21
Gambia                                     26.24
Georgia                                    41.57
Germany                                   510.88
Ghana                                      23.47
Greece                                    138.19
Greenland                                5136.92
Grenada                                   189.50
Guatemala                                  56.86
Guinea                                      6.53
Guinea-Bissau                               1.17
Guyana                                      1.76
Haiti                                     100.72
Honduras                                   17.33
Hungary                                   192.04
Iceland                                   713.14
India                                     383.83
Indonesia                                  78.45
Iran                                      751.72
Iraq                                     2118.44
Ireland                                   431.64
Israel                                   4191.72
Italy                                     319.85
Jamaica                                   126.27
Japan                                     382.47
Jordan                                   2159.55
Kazakhstan                                731.77
Kenya                                      58.47
Kiribati                                  584.02
Kuwait                                 158791.25
Kyrgyzstan                                 74.04
Laos                                       14.56
Latvia                                     18.36
Lebanon                                  1336.57
Lesotho                                  1126.72
Liberia                                     0.88
Libya                                    2809.95
Liechtenstein                             234.68
Lithuania                                  54.31
Luxembourg                                740.84
Madagascar                                  3.57
Malawi                                      6.22
Malaysia                                  150.51
Maldives                                25318.85
Mali                                        5.00
Malta                                   40115.13
Marshall Islands                          162.22
Mauritania                                141.07
Mauritius                                1062.33
Mexico                                     72.70
Moldova                                   149.88
Mongolia                                   32.93
Montenegro                                 27.42
Montserrat                                 87.20
Morocco                                    74.48
Mozambique                                  2.15
Namibia                                     6.28
Nepal                                      27.24
Netherlands                              2602.95
New Caledonia                              59.51
New Zealand                                29.82
Nicaragua                                  15.14
Niger                                      24.05
Nigeria                                    58.60
Niue                                        4.06
North Korea                               100.66
North Macedonia                            72.95
Norway                                     36.01
Oman                                   314045.20
Pakistan                                  521.51
Palau                                      53.81
Panama                                     32.69
Papua New Guinea                            2.27
Paraguay                                    4.94
Peru                                        7.69
Philippines                               213.72
Poland                                    298.72
Portugal                                  111.81
Romania                                    95.05
Russia                                     21.33
Rwanda                                     52.27
Saint Kitts and Nevis                     211.43
Saint Lucia                               243.83
Samoa                                      15.88
Sao Tome and Principe                      31.88
Saudi Arabia                             7535.37
Senegal                                    14.44
Serbia                                    175.63
Seychelles                                165.30
Sierra Leone                                4.89
Singapore                               31193.17
Slovakia                                  156.50
Slovenia                                   90.91
Solomon Islands                             1.16
Somalia                                     0.92
South Africa                              234.55
South Korea                               898.08
South Sudan                                 2.17
Spain                                     117.95
Sri Lanka                                  90.71
Sudan                                      10.82
Suriname                                    1.66
Sweden                                     11.82
Switzerland                               246.95
Syria                                     478.57
Taiwan                                   1082.59
Tajikistan                                205.78
Tanzania                                    3.69
Thailand                                  132.46
Togo                                       20.75
Tonga                                     214.67
Trinidad and Tobago                      1504.51
Tunisia                                   443.12
Turkey                                    190.99
Turkmenistan                              147.82
Turks and Caicos Islands                  353.02
Tuvalu                                    132.69
Uganda                                     19.98
Ukraine                                   135.17
United Arab Emirates                     6141.39
United Kingdom                            949.97
United States                             147.48
Uruguay                                    38.47
Uzbekistan                                333.01
Vanuatu                                     5.11
Venezuela                                  20.83
Vietnam                                   214.00
Yemen                                     182.79
Zambia                                      1.71
Zimbabwe                                    6.34
```

## Sources that provide information on forest CO₂ adsorption rates:
1. How Much CO2 Does A Tree Absorb? (https://onetreeplanted.org/blogs/stories/how-much-co2-does-tree-absorb)
2. https://www.pnas.org/doi/10.1073/pnas.0702737104 (https://www.pnas.org/doi/10.1073/pnas.0702737104)
3. Planting trees is not a solution (https://www.carbonindependent.org/76.html)
4. How much CO2 does a tree absorb? Let’s get carbon curious! (https://ecotree.green/en/how-much-co2-does-a-tree-absorb)
5. STATE OF THE WORLD'S FORESTS 2001 (https://www.fao.org/4/y0900e/y0900e06.htm)

## Data for the calculations:
1. CO2 emissions by country - GitHub - owid/co2-data: Data on CO2 and greenhouse gas emissions by Our World in Data (https://github.com/owid/co2-data)
2. Forest share of land area by country - https://ourworldindata.org/grapher/forest-area-as-share-of-land-area.csv?v=1&csvType=full&useColumnShortNames=true (https://ourworldindata.org/grapher/forest-area-as-share-of-land-area.csv?v=1&csvType=full&useColumnShortNames=true)
3. Land area by country - https://www.cia.gov/the-world-factbook/field/area/country-comparison/ (https://www.cia.gov/the-world-factbook/field/area/country-comparison/)
