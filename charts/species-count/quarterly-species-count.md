

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
    dv.span(`**Species Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else if (targetmonth === 1) {
    targetyear2 = targetyear - 1;
    targetmonth2 = targetmonth - 1;
    targetmonth3 = targetmonth + 11;
    dv.span(`**Species Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else {
    targetyear2 = targetyear;
    targetmonth2 = targetmonth - 1;
    targetmonth3 = targetmonth - 2;
    dv.span(`**Species Count for ${targetmonth3 + 1}-${targetmonth + 1}/${targetyear}**`);
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

// Initialize an object to count species occurrences
const speciesCounts = {};

// Iterate over each bird to count species occurrences
birdsthismonth.forEach(bird => {
    const title = bird.file.name; // Get the note title
    const species = title.slice(-4); // Get the last four characters for species

    // Initialize the species count if it doesn't exist
    if (!speciesCounts[species]) {
        speciesCounts[species] = 0;
    }

    // Increment the count for the species
    speciesCounts[species]++;
});

// Generate unique colors for all species
const sortedSpecies = Object.keys(speciesCounts).sort();
const colorMapping = generateUniqueColors(sortedSpecies.length);

// Create a map to associate species with their color
const speciesColorMap = sortedSpecies.reduce((acc, species, index) => {
    acc[species] = colorMapping[index];
    return acc;
}, {});

// Prepare data for the chart
const specieslist = sortedSpecies; // Get sorted list of species

// Create datasets based on the species counts
const datasets = [{
    data: specieslist.map(species => speciesCounts[species]), // Correctly map counts for each species
    backgroundColor: specieslist.map(species => speciesColorMap[species] || '#CCCCCC'), // Color for each species
}];

const chartData = {
    type: 'bar',
    data: {
        labels: specieslist, // Species on x-axis
        datasets: datasets, // One dataset containing counts for all species
    },
    options: {
        responsive: true,
        scales: {
            x: {
                stacked: false, // Don't stack bars
            },
            y: {
                stacked: false, // Don't stack the bars
                beginAtZero: true,
            },
        },
        plugins: {
            legend: {
                display: false, // Disable the legend (the label at the top of the graph)
            },
        },
    },
};

// Render the chart
window.renderChart(chartData, this.container);

// Function to generate unique colors
function generateUniqueColors(count) {
    const colors = [];
    for (let i = 0; i < count; i++) {
        const hue = (i * (360 / count)) % 360; // Distribute hues evenly
        colors.push(`hsl(${hue}, 70%, 50%)`);
    }
    return colors;
}
```
