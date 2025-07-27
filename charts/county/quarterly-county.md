
```dataviewjs
const targetyear = moment().year();
const targetmonth = moment().month(); // December = 11 (month is zero-indexed)

let targetyear2 = 0;
let targetmonth2 = 0;
let targetmonth3 = 0;


// If the month is Jan or Feb, we need to go back a year:
if (targetmonth === 0) {
	targetyear2 = targetyear - 1;
	targetmonth2 = targetmonth + 11;
	targetmonth3 = targetmonth + 10;
	dv.span(`**City + County Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else if (targetmonth === 1) {
	targetyear2 = targetyear - 1;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth + 11;
	dv.span(`**City and County Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else {
	targetyear2 = targetyear;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth - 2;
		dv.span(`**City and County Count for ${targetmonth3 + 1}-${targetmonth + 1}/${targetyear}**`);
}


// Helper function to get the year of a date field, returning null if the field is empty or invalid
function getYearFromDateField(dateField) {
    if (!dateField) return null;
    const date = new Date(dateField);
    return isNaN(date.getTime()) ? null : date.getFullYear();
}

// Filter birds for the target year based only on the Outtake year
const birds = dv.pages('"RARE Birds"')
    .where(bird => {
        const outtakeYear = getYearFromDateField(bird.Outtake);  // Get the outtake year
        // Only include birds where the Outtake year is the target year or null
        return outtakeYear === targetyear || outtakeYear === targetyear2 || outtakeYear === null;
    });

let birdsthismonth = [];

// Iterate over each bird note
birds.forEach((bird) => {
    const intakeYear = new Date(bird.Intake).getFullYear();
    const intakeMonth = new Date(bird.Intake).getMonth();
    const outtakeYear = getYearFromDateField(bird.Outtake);  // Get the outtake year
    const outtakeMonth = bird.Outtake ? new Date(bird.Outtake).getMonth() : null;

    const isOuttakeNull = !bird.Outtake || isNaN(new Date(bird.Outtake).getTime());
    const hasUnknownStatus = bird.Status && Object.values(bird.Status).includes("Unknown");

    if (
        ((intakeMonth === targetmonth || intakeMonth == targetmonth2 || intakeMonth === targetmonth3)  && (intakeYear === targetyear || intakeYear === targetyear2)) ||
        (isOuttakeNull && !hasUnknownStatus) ||
        ((outtakeMonth === targetmonth || outtakeMonth === targetmonth2 || outtakeMonth === targetmonth3) && (outtakeYear === targetyear || outtakeYear === targetyear2))
    ) {
        // Skip notes with specific paths
        if (bird.file.path.includes("RARE Birds/Ed Birds") || bird.file.path.includes("RARE Birds/Species") || bird.file.path.includes("2221 RLHA")) {
            return; // Skip this note
        }
        birdsthismonth.push(bird);
    }
});

// Initialize an object to store city counts by county
const cityCountsByCounty = {};

// Process notes and count cities by county
birdsthismonth.forEach(note => {

  // Check if Location exists and is an array
  if (note.file.frontmatter.Location && Array.isArray(note.file.frontmatter.Location)) {
    // Extract the county name if it exists
    const countyName = note.file.frontmatter.Location.find(loc => loc.includes("County"));
    if (countyName) {
      // Initialize the county's city counts if it doesn't exist
      cityCountsByCounty[countyName] = cityCountsByCounty[countyName] || {};

      // Extract the city name (the one that does not contain "County")
      const cityName = note.file.frontmatter.Location.find(loc => !loc.includes("County"));

      // Handle the city name
      if (cityName) {
        cityCountsByCounty[countyName][cityName] = (cityCountsByCounty[countyName][cityName] || 0) + 1;
      } else {
        // Handle "Unknown City" case
        const unknownCity = `Unknown ${note.file.frontmatter.Location[0]} City`;
        cityCountsByCounty[countyName][unknownCity] = (cityCountsByCounty[countyName][unknownCity] || 0) + 1;
      }
    }
  }
});

// Adjusted base colors for each county (neon-like colors that are easy to see on dark and light modes)
const baseColors = {
    "Benton County": "rgba(0, 255, 255, 0.9)", // Neon Cyan
    "Blackhawk County": "rgba(255, 20, 147, 0.9)", // Neon Pink
    "Cerro Gordo County": "rgba(255, 255, 0, 0.9)", // Neon Yellow
    "Clinton County": "rgba(0, 255, 0, 0.9)", // Neon Green
    "Cedar County": "rgba(255, 165, 0, 0.9)", // Neon Orange
    "Clayton County": "rgba(0, 0, 255, 0.9)", // Neon Blue
    "Dallas County": "rgba(138, 43, 226, 0.9)", // Neon Purple
    "Dubuque County": "rgba(255, 0, 0, 0.9)", // Neon Red
    "Des Moines County": "rgba(255, 105, 180, 0.9)", // Neon Hot Pink
    "Henry County": "rgba(255, 69, 0, 0.9)", // Neon Orange-Red
    "Jackson County": "rgba(32, 178, 170, 0.9)", // Neon Medium Turquoise
    "Jasper County": "rgba(0, 255, 255, 0.9)", // Neon Cyan
    "Johnson County": "rgba(255, 99, 132, 0.9)", // Neon Light Red
	"Jefferson County": "rgba(255, 105, 180, 0.9)", // Neon Hot Pink
    "Jones County": "rgba(255, 69, 0, 0.9)", // Neon Orange-Red
    "Lee County": "rgba(255, 160, 122, 0.9)", // Neon Light Salmon
    "Linn County": "rgba(0, 0, 255, 0.9)", // Neon Blue
    "Louisa County": "rgba(138, 43, 226, 0.9)", // Neon Purple
    "Mahaska County": "rgba(255, 165, 0, 0.9)", // Neon Orange
    "Marshall County": "rgba(255, 20, 147, 0.9)", // Neon Pink
    "Monroe County": "rgba(0, 255, 0, 0.9)", // Neon Green
    "Muscatine County": "rgba(0, 0, 255, 0.9)", // Neon Blue
    "Pottawattamie County": "rgba(255, 69, 0, 0.9)", // Neon Red-Orange
    "Polk County": "rgba(75, 0, 130, 0.9)", // Neon Indigo
    "Scott County": "rgba(255, 255, 0, 0.9)", // Neon Yellow
    "Story County": "rgba(255, 0, 0, 0.9)", // Neon Red
    "Wapello County": "rgba(32, 178, 170, 0.9)", // Neon Medium Turquoise
    "Washington County": "rgba(255, 99, 132, 0.9)", // Neon Red
    "Webster County": "rgba(138, 43, 226, 0.9)", // Neon Purple
    "Woodbury County": "rgba(0, 255, 255, 0.9)", // Neon Cyan
    "Van Buren County": "rgba(255, 20, 147, 0.9)" // Neon Pink
};

// Default fallback color in case the base color is missing or undefined
const defaultColor = "rgba(255, 0, 255, 0.9)"; // Neon Magenta (for undefined counties)

// Function to generate gradient colors for cities in each county
function getGradientColors(baseColor, cityIndex, totalCities) {
    if (!baseColor) return 'rgba(255, 223, 0, 0.9)'; // Fallback color with higher alpha

    // Ensure baseColor is in a valid format (rgba or rgb)
    const colorMatch = baseColor.match(/rgba?\((\d+), (\d+), (\d+),? ?(\d?\.?\d*)\)/);
    if (!colorMatch) return 'rgba(255, 223, 0, 0.9)'; // Invalid color format fallback

    // Parse color values
    const r = parseInt(colorMatch[1]);
    const g = parseInt(colorMatch[2]);
    const b = parseInt(colorMatch[3]);
    const a = colorMatch[4] || 1; // Use full opacity if alpha is missing

    // Convert RGB to HSL for better manipulation
    const hslColor = rgbToHsl([r, g, b]);

    // Calculate a slight variation in lightness based on the cityIndex
    const lightnessVariation = cityIndex / totalCities; // Varies from 0 to 1
    
    // Create a new HSL color with adjusted lightness
    const newHslColor = {
        h: hslColor.h,
        s: hslColor.s, // Keep the same saturation
        l: hslColor.l + (lightnessVariation * 0.4 - 0.2) // Adjust lightness based on the city index
    };

    // Convert the new HSL color back to RGB and return it
    return hslToRgb(newHslColor, a); // Pass alpha along to maintain transparency
}


// Function to generate the gradient colors for each city within a county
function applyGradientColor(baseColor, cityIndex, cityNames) {
    const gradientColors = cityNames.map((city, index) => {
        return getGradientColors(baseColor, index, cityNames.length); // Generate gradient for each city
    });
    return gradientColors;
}

// Helper function to convert RGB to HSL
function rgbToHsl(rgb) {
    const r = rgb[0] / 255;
    const g = rgb[1] / 255;
    const b = rgb[2] / 255;
    const max = Math.max(r, g, b);
    const min = Math.min(r, g, b);
    const delta = max - min;
    const h = (max === min ? 0 : max === r ? (g - b) / delta + (g < b ? 6 : 0) : max === g ? (b - r) / delta + 2 : (r - g) / delta + 4) * 60;
    const l = (max + min) / 2;
    const s = max === min ? 0 : delta / (1 - Math.abs(2 * l - 1));

    return { h, s, l };
}

// Helper function to convert HSL back to RGB
function hslToRgb(hsl) {
    const { h, s, l } = hsl;
    const c = (1 - Math.abs(2 * l - 1)) * s;
    const x = c * (1 - Math.abs((h / 60) % 2 - 1));
    const m = l - c / 2;
    let r = 0, g = 0, b = 0;

    if (0 <= h && h < 60) {
        r = c; g = x; b = 0;
    } else if (60 <= h && h < 120) {
        r = x; g = c; b = 0;
    } else if (120 <= h && h < 180) {
        r = 0; g = c; b = x;
    } else if (180 <= h && h < 240) {
        r = 0; g = x; b = c;
    } else if (240 <= h && h < 300) {
        r = x; g = 0; b = c;
    } else {
        r = c; g = 0; b = x;
    }

    return `rgb(${Math.round((r + m) * 255)}, ${Math.round((g + m) * 255)}, ${Math.round((b + m) * 255)})`;
}


// Prepare data for the chart
const datasets = [];
const countyNames = []; // Array to hold all county names
const cityNamesSet = new Set(); // Set to hold all unique city names
const countyColors = {}; // Map to store the color for each county

// Loop through the counties and cities to gather the necessary data
for (const [county, cityCounts] of Object.entries(cityCountsByCounty)) {
    countyNames.push(county); // Add the county name to the labels
    // Use county color or fallback to a default color
    countyColors[county] = baseColors[county] || defaultColor; // Ensure valid color is assigned

    // Sort the cities alphabetically, with "Unknown City" last
    const sortedCities = Object.keys(cityCounts).sort((a, b) => {
        if (a.includes("Unknown") || b.includes("Unknown")) {
            return a.includes("Unknown") ? 1 : -1; // "Unknown City" comes last
        } else {
            return a.localeCompare(b); // Sort alphabetically
        }
    });

    // For each city in this county, add data to the city dataset
    sortedCities.forEach(city => {
        cityNamesSet.add(city); // Add city name to the set of city names
    });
}

const cityNames = Array.from(cityNamesSet).sort(); // Sort city names alphabetically
countyNames.sort(); // Sort county names

// Now create the datasets using the county base colors and gradient colors for each city
cityNames.forEach((city, cityIndex) => {
    const cityData = countyNames.map(county => {
        const cityCount = cityCountsByCounty[county] && cityCountsByCounty[county][city] || 0;
        return cityCount;
    });

    // Use getGradientColors to create a gradient for the cities within a county
    const backgroundColors = countyNames.map(county => {
        const baseColor = countyColors[county]; // Get the base color for the county
        return getGradientColors(baseColor, cityIndex, cityNames.length); // Apply gradient for each city
    });

    datasets.push({
        label: city,
        data: cityData,
        backgroundColor: backgroundColors, // Ensure gradient background is used
        borderWidth: 0
    });
});

// Chart.js data structure
const chartData = {
    type: 'bar',
    data: {
        labels: countyNames, // Sorted county names as labels on the x-axis
        datasets: datasets // Datasets for each city, with data for all counties
    },
    options: {
        scales: {
            x: {
                stacked: true, // Stack the bars for each county
                ticks: {
                    display: true
                }
            },
            y: {
                stacked: true, // Stack the bars
                beginAtZero: true
            }
        },
        plugins: {
            legend: {
                display: false // Remove the legend
            },
            tooltip: {
                callbacks: {
                    label: (tooltipItem) => {
                        const cityLabel = tooltipItem.dataset.label || '';
                        const count = tooltipItem.raw || 0;
                        return `${cityLabel}: ${count}`;
                    }
                }
            }
        }
    }
};

window.renderChart(chartData, this.container);
```
