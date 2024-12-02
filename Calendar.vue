<template>
  <div class="p-4">
    <!-- Search Categories Section -->
    <div class="mb-6">
      <!-- <h1 class="text-xl mb-1  text-gray-500">Search Categories</h1> -->
      <div class="flex justify-center items-center gap-4 ">
        <!-- Input Fields and Dropdowns for Search Categories -->
        <div class="flex flex-wrap items-center justify-center text-gray-600 gap-4">
          <div v-for="folder in folders" :key="folder.name" class="flex flex-col">
            <label :for="folder.name" class="font-medium text-sm">{{ folder.name }}</label>
            <!-- Filter By as a Text Input -->
            <input
              v-if="folder.name === 'Filter By'"
              :id="folder.name"
              v-model="folder.value"
              type="text"
              :placeholder="folder.placeholder"
              class="border rounded px-1 py-1 w-40 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            <!-- Other Fields as Dropdowns -->
            <select
              v-else
              :id="folder.name"
              v-model="folder.value"
              class="border rounded px-1 py-1 w-40 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option disabled value="">Select {{ folder.name }}</option>
              <option v-for="option in folder.options" :key="option" :value="option">
                {{ option }}
              </option>
            </select>
          </div>
        </div>
        <!-- Refresh Button -->
        <button
          @click="refreshPage"
          class="bg-blue-400 text-white font-semibold rounded px-5 py-1 hover:bg-blue-700"
        >
          Refresh
        </button>
      </div>
    </div>

    <!-- Calendar Section -->
    <div class="border rounded-lg shadow-md p-2 bg-white">
      <h2 class="text-xm font-bold transform translate-y-24 text-gray-700">Month days:</h2>
      <div class="flex justify-between items-center mb-2">
        <!-- Left Navigation Button with Start Date -->
        <div class="flex items-center">
          <span class="text-xs mr-1 text-gray-500">{{ formattedStartDate }}</span>
          <button
            @click="slideLeft"
            class="bg-blue-400 text-white rounded-full w-7 h-10 flex items-center justify-center hover:bg-blue-800"
            :disabled="currentPage === 0"
          >
            &lt;
          </button>
        </div>

        <!-- Right Navigation Button with End Date -->
        <div class="flex items-center">
          <button
            @click="slideRight"
            class="bg-blue-400 text-white rounded-full w-7 h-10 flex items-center justify-center hover:bg-blue-800"
            :disabled="currentPage === pages.length - 1"
          >
            &gt;
          </button>
          <span class="text-xs ml-1 text-gray-500">{{ formattedEndDate }}</span>
        </div>
      </div>

      <!-- Calendar Content -->
      <div id="calendar" class="flex flex-wrap gap-0 overflow-hidden m-16 "></div>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "SearchCategoryCalendar",
  data() {
    return {
      blockSize: 20, // Size of calendar blocks
      daysPerPage: 40, // Number of days to display on a single page
      overlapDays: 1, // Overlap days from the next page
      pages: [], // Array of pages with overlapping data
      currentPage: 0, // Currently visible page index
      folders: [
        { name: "Filter By", value: "", placeholder: "Product ID or Name" },
        {
          name: "Category",
          value: "",
          options: ["Electronics", "Clothing", "Books", "Groceries"],
        },
        {
          name: "Vendors",
          value: "",
          options: ["Vendor A", "Vendor B", "Vendor C"],
        },
        {
          name: "Shipping Location",
          value: "",
          options: ["New York", "Los Angeles", "Chicago"],
        },
        {
          name: "Restock Location",
          value: "",
          options: ["Warehouse A", "Warehouse B", "Warehouse C"],
        },
      ],
    };
  },
  computed: {
    currentDays() {
      return this.pages[this.currentPage] || [];
    },
    formattedStartDate() {
      const startDate = this.currentDays[0]?.date;
      return startDate ? this.formatDate(startDate, "start date") : "";
    },
    formattedEndDate() {
      const endDate = this.currentDays[this.currentDays.length - 1]?.date;
      return endDate ? this.formatDate(endDate, "end date") : "";
    },
  },
  methods: {
    formatDate(date, type) {
      const options = { year: "numeric", month: "long", day: "2-digit" };
      const formattedDate = new Intl.DateTimeFormat("en-GB", options).format(date);
      return type === "start date" ? `Start date: ${formattedDate}` : `End date: ${formattedDate}`;
    },
    generatePages() {
      // Generate dummy data for the calendar
      const dummyData = [];
      const startDate = new Date(2023, 0, 1); // Start date as January 1, 2024
      for (let i = 0; i < 3650; i++) {
        dummyData.push({
          date: new Date(startDate.getFullYear(), startDate.getMonth(), startDate.getDate() + i),
          change: Math.random() * 10 - 5, // Random change between -5 and 5
        });
      }

      // Create pages with overlapping days
      let currentPage = [];
      dummyData.forEach((day) => {
        currentPage.push(day);

        if (currentPage.length === this.daysPerPage) {
          this.pages.push([...currentPage]);
          currentPage = currentPage.slice(-this.overlapDays);
        }
      });

      if (currentPage.length > 0) {
        this.pages.push(currentPage);
      }
    },
    renderCalendar() {
  const calendarContainer = d3.select("#calendar");
  calendarContainer.selectAll("*").remove(); // Clear previous content

  const days = this.currentDays;

  // Render blocks using D3.js for each date
  const dayElements = calendarContainer
    .selectAll(".day")
    .data(days)
    .enter()
    .append("div")
    .attr("class", "day")
    .style("width", `${this.blockSize}px`)  // Set width for each block
    .style("height", `${this.blockSize}px`) // Set height for each block
    .style("display", "inline-block")       // Display each block inline
    .style("margin", "1px")                 // Add a little margin between blocks
    .style("background-color", "#f5f5f5") // No background color for blocks
    .style("color", "blue")                 // Blue text color for dates
    .style("text-align", "center")          // Center the text horizontally
    .style("line-height", `${this.blockSize}px`) // Center the text vertically
    .attr("title", "")  // Tooltip on hover
    .text((d) => d.date.getDate());  // Display the day of the month
}
,
    slideLeft() {
      if (this.currentPage > 0) {
        this.currentPage -= 1;
        this.renderCalendar();
      }
    },
    slideRight() {
      if (this.currentPage < this.pages.length - 1) {
        this.currentPage += 1;
        this.renderCalendar();
      }
    },
    refreshPage() {
      this.currentPage = 0;
      this.folders.forEach((folder) => (folder.value = "")); // Clear all inputs and dropdowns
      this.renderCalendar();
    },
  },
  mounted() {
    this.generatePages();
    this.renderCalendar();
  },
};
</script>

<style scoped>
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
