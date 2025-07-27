
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
	dv.span(`**Injury Events for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else if (targetmonth === 1) {
	targetyear2 = targetyear - 1;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth + 11;
	dv.span(`**Injury Events for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else {
	targetyear2 = targetyear;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth - 2;
		dv.span(`**Injury Events for ${targetmonth3 + 1}-${targetmonth + 1}/${targetyear}**`);
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

// Initialize an object to store injury counts by group
const injuryCounts = {};

// Define groups and their respective injuries
const injuryGroups = {
	Displaced: ["Displaced from nest"],
	Exhaustion: ["Exhaustion"], 
	Caught: ["Caught in fishing line", "Caught in plastic netting", "Caught in trap", "Trapped in barb wire", "Stuck in tree", "Trapped", "Caught in fence", "Stuck in chimney"],
	Fell: ["Fell from tree", "Fell off deck", "Drowned"],
	Strike: ["Hit ceiling fan", "Ran into fence", "Ran into windmill", "Window Strike"], 
	Puncture: ["Bitten", "Attacked", "Shot", "Puncture"],
	HBC: ["HBC"],
	Poisoned: ["Poisoned"],
	Infection: ["Infection"],
	Cold: ["Frozen", "Ice Storm"], 
	Hot: ["Fire", "Heat stress"],
	"Unable to fly": ["Unable to fly"]
};

// Define base colors for each group
const baseColors = {
	Caught: "rgba(255, 165, 0, 0.7)", // Orange - A bright orange that suggests a sense of urgency and being trapped.
	Displaced: "rgba(102, 153, 255, 0.7)", // Light Blue - A calming light blue that represents a sense of being lost and adrift.
	Exhaustion: "rgba(255, 204, 153, 0.7)", // Peach - A soft, warm peach that evokes a sense of calm and relaxation, but with a hint of weariness. 
	Fell: "rgba(255, 0, 0, 0.7)", // Red - A bold red that symbolizes the impact of falling and the feeling of pain.
	Strike: "rgba(255, 255, 0, 0.7)", // Yellow - A bright yellow that represents the suddenness and intensity of a strike.
	Puncture: "rgba(153, 102, 255, 0.7)", // Purple - A vibrant purple that evokes a sense of mystery and danger.
	HBC: "rgba(153, 0, 76, 0.7)", // A deep, rich purple that evokes a sense of danger and urgency. 
	Poisoned: "rgba(128, 0, 0, 0.7)", // Dark Red - A dark red that symbolizes the danger and toxicity of poison.
	Infection: "rgba(0, 255, 0, 0.7)", // Green - A bright green that suggests the growth and spread of infection.
	Cold: "rgba(0, 191, 255, 0.7)", // Light Blue - A cool light blue that evokes the feeling of coldness and numbness.
	Hot: "rgba(255, 69, 0, 0.7)", // Orange - A fiery orange that represents the feeling of heat and burning.
	"Unable to fly": "rgba(247, 65, 47, 0.7)",
    Misc: "rgba(128, 128, 128, 0.7)" // Grey for Misc
};

// Process notes and count injuries by group
birdsthismonth.forEach(note => {

    const injuries = note.file.frontmatter.InjuryEvent;

    if (!injuries || !Array.isArray(injuries) || injuries.length === 0) return;

    // Count injuries, excluding certain items
    injuries.forEach(injury => {
        let injuryStr = String(injury);

        // Flag to check if the injury matched any group
        let foundGroup = false;

        // Loop through all the injury groups
        for (const [group, items] of Object.entries(injuryGroups)) {
            // Only proceed if the items is an array (predefined injury group)
            if (Array.isArray(items)) {
                // Check if any item in the group is part of the injury string
                items.forEach(item => {
                    if (injuryStr.includes(item)) {
                        injuryCounts[group] = injuryCounts[group] || {};
                        injuryCounts[group][item] = (injuryCounts[group][item] || 0) + 1;
                        foundGroup = true;
                    }
                });
            }
        }

        // If no group was matched, consider it as Misc
        if (!foundGroup) {
            injuryCounts.Misc = injuryCounts.Misc || {};
            injuryCounts.Misc[injuryStr] = (injuryCounts.Misc[injuryStr] || 0) + 1;
        }
    });
});

// Prepare data for the chart
const datasets = [];
const injuryGroupNames = Object.keys(injuryGroups); // Get the group names for x-axis labels

// Create datasets for each injury type within their respective groups, sorted alphabetically
injuryGroupNames.forEach((group) => {
    // Sort the injuries alphabetically for each group
    const injuries = injuryGroups[group] ? [...injuryGroups[group]].sort() : [];

    injuries.forEach((injury, index) => {
        const totalCount = injuryCounts[group]?.[injury] || 0; // Total count for the injury

        // Create a dataset for this specific injury
        datasets.push({
            label: injury, // Use injury name as label
            data: injuryGroupNames.map(g => (g === group ? totalCount : 0)), // Count for this group, 0 for others
            backgroundColor: getGradientColor(baseColors[group], index, injuries.length), // Unique color for each injury
            stack: 'stack1' // Use a single stack identifier
        });
    });
});


// Special handling for Misc injuries (if any)
const miscInjuries = injuryCounts.Misc ? Object.keys(injuryCounts.Misc) : [];
if (miscInjuries.length > 0) {
    // Add Misc to the labels and its dataset
    injuryGroupNames.push('Misc'); // Add Misc to the labels for x-axis

    miscInjuries.forEach((injury, index) => {
        const totalCount = injuryCounts.Misc[injury]; // Total count for Misc injury

        // Create a dataset for Misc injuries
        datasets.push({
            label: injury, // Use injury name as label
            data: injuryGroupNames.map(g => (g === 'Misc' ? totalCount : 0)), // Count for Misc, 0 for others
            backgroundColor: baseColors.Misc, // Grey color for Misc
            stack: 'stack1' // Use a single stack identifier
        });
    });
}


// Prepare x-axis labels (now includes Misc if it has data)
const labels = injuryGroupNames; // Labels include Misc if it has injuries

// Function to generate a gradient color based on the base color and index
function getGradientColor(baseColor, index, total) {
    const alpha = (index + 1) / total; // Compute alpha based on index
    return baseColor.replace(/0\.7/, alpha); // Replace alpha in the base color
}

// Assuming you are using a charting library, here’s how you might render it
const chartData = {
    type: 'bar',
    data: {
        labels: labels, // Use the updated labels (now includes Misc if it has data)
        datasets: datasets
    },
    options: {
        scales: {
            x: {
                stacked: true, // Enable stacking
                ticks: {
                    display: true // Enable labels display for x-axis
                }
            },
            y: {
                stacked: true // Enable stacking
            }
        },
        plugins: {
            legend: {
                display: false // Turn off the legend
            }
        }
    }
};

// Render the chart (replace this with your chart rendering code)
window.renderChart(chartData, this.container);
```
