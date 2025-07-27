
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
	dv.span(`**Status Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else if (targetmonth === 1) {
	targetyear2 = targetyear - 1;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth + 11;
	dv.span(`**Status Count for ${targetmonth3 + 1}/${targetyear2} - ${targetmonth + 1}/${targetyear}**`);
} else {
	targetyear2 = targetyear;
	targetmonth2 = targetmonth - 1;
	targetmonth3 = targetmonth - 2;
		dv.span(`**Status Count for ${targetmonth3 + 1}-${targetmonth + 1}/${targetyear}**`);
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

// Initialize an object to count status occurrences by year
const statusCounts = {};

// Define color mapping for each status with updated Barn color
const colorMapping = {
    "Unknown": '#808080', // gray
    "Dead on Arrival (DOA)": '#45226D',
    "Died": '#5C2D91', // dark purple
    "Euthanized": '#004080', // dark blue
    "Clinic": '#FF0000', // red
    "Barn": '#FFD700', // new shade of yellow
    "Transferred": '#04F06A',
    "Released": '#008000' // green
};

// Order of statuses from bottom to top
const statusOrder = [
    "Unknown",
    "Dead on Arrival (DOA)",
    "Died",
    "Euthanized",
    "Clinic",
    "Barn",
    "Transferred",
    "Released"
];

// Iterate over each note and extract year and status
birdsthismonth.forEach(note => {
    const title = note.file.name; // Get the note title

    const status = note.Status; // Get the status of the note

    // Initialize the status count if it doesn't exist
    if (!statusCounts[status]) {
        statusCounts[status] = 0;
    }

    // Increment the count for the status
    statusCounts[status]++;
});

// Prepare data for the chart
const statuslist = statusOrder.filter(status => statusCounts[status]); // Get sorted list of statuses that have counts

// Create datasets based on the status counts
const datasets = [{
    data: statuslist.map(status => statusCounts[status] || 0), // Correctly map counts for each status
    backgroundColor: statuslist.map(status => colorMapping[status] || '#CCCCCC'), // Color for each status
}];

// Create chart data structure
const chartData = {
    type: 'bar',
    data: {
        labels: statuslist, // Statuses on x-axis
        datasets: datasets, // One dataset containing counts for all statuses
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
```
