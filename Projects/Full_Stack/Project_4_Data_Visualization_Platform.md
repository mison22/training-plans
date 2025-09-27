# Project 4: Data Visualization Platform
*Full Stack Intensive Plan - Weeks 7-8*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.md)** | 🏠 **[Back to README](../../README.md)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 7-8)  
**Tech Stack**: Angular + TypeScript + Node.js + Express + MongoDB + D3.js  
**Focus**: Advanced frontend frameworks, data visualization, and full-stack production deployment  

### Learning Objectives
- Master Angular with TypeScript for enterprise-scale applications
- Build Node.js backend with Express and MongoDB
- Implement advanced data visualization with D3.js
- Handle complex state management and routing
- Deploy full-stack applications to production

---

## 📊 Database Schema Design (MongoDB)

### Data Sources Collection
```javascript
{
  _id: ObjectId,
  name: String,
  type: String, // 'csv', 'api', 'database', 'manual'
  description: String,
  connection_config: {
    url: String,
    headers: Object,
    query_params: Object,
    authentication: {
      type: String, // 'none', 'basic', 'bearer', 'api_key'
      credentials: Object
    }
  },
  data_schema: {
    fields: [{
      name: String,
      type: String, // 'string', 'number', 'date', 'boolean'
      required: Boolean,
      description: String
    }]
  },
  owner_id: ObjectId,
  is_public: Boolean,
  created_at: Date,
  updated_at: Date
}
```

### Datasets Collection
```javascript
{
  _id: ObjectId,
  name: String,
  description: String,
  data_source_id: ObjectId,
  data: [Object], // Array of data records
  metadata: {
    total_records: Number,
    last_updated: Date,
    file_size: Number,
    columns: [String],
    sample_data: [Object] // First 5 records for preview
  },
  owner_id: ObjectId,
  tags: [String],
  is_public: Boolean,
  created_at: Date,
  updated_at: Date
}
```

### Charts Collection
```javascript
{
  _id: ObjectId,
  name: String,
  description: String,
  dataset_id: ObjectId,
  chart_type: String, // 'bar', 'line', 'pie', 'scatter', 'heatmap', 'treemap'
  configuration: {
    x_axis: {
      field: String,
      label: String,
      type: String // 'category', 'time', 'number'
    },
    y_axis: {
      field: String,
      label: String,
      type: String
    },
    color_field: String,
    size_field: String,
    filters: [{
      field: String,
      operator: String, // 'eq', 'gt', 'lt', 'contains'
      value: Mixed
    }],
    aggregation: {
      type: String, // 'count', 'sum', 'avg', 'min', 'max'
      field: String
    },
    styling: {
      colors: [String],
      width: Number,
      height: Number,
      theme: String
    }
  },
  dashboard_id: ObjectId,
  position: {
    x: Number,
    y: Number,
    width: Number,
    height: Number
  },
  owner_id: ObjectId,
  is_public: Boolean,
  created_at: Date,
  updated_at: Date
}
```

### Dashboards Collection
```javascript
{
  _id: ObjectId,
  name: String,
  description: String,
  layout: {
    grid_size: {
      columns: Number,
      rows: Number
    },
    breakpoints: {
      mobile: Object,
      tablet: Object,
      desktop: Object
    }
  },
  charts: [ObjectId], // References to charts
  filters: [{
    id: String,
    type: String, // 'date_range', 'dropdown', 'checkbox'
    field: String,
    label: String,
    options: [String] // For dropdown/checkbox
  }],
  sharing: {
    is_public: Boolean,
    access_level: String, // 'view', 'edit'
    password: String,
    expires_at: Date
  },
  owner_id: ObjectId,
  created_at: Date,
  updated_at: Date
}
```

---

## 🛠️ Frontend Components (Angular + TypeScript)

### Dashboard Component
```typescript
// components/dashboard/dashboard.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { Subject, takeUntil } from 'rxjs';

import { DashboardService } from '../../services/dashboard.service';
import { ChartService } from '../../services/chart.service';
import { Dashboard, Chart, FilterValue } from '../../models';

@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
  styleUrls: ['./dashboard.component.scss']
})
export class DashboardComponent implements OnInit, OnDestroy {
  dashboard: Dashboard | null = null;
  charts: Chart[] = [];
  filters: FilterValue[] = [];
  loading = true;
  error: string | null = null;
  
  private destroy$ = new Subject<void>();

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private dashboardService: DashboardService,
    private chartService: ChartService
  ) {}

  ngOnInit(): void {
    this.route.params
      .pipe(takeUntil(this.destroy$))
      .subscribe(params => {
        this.loadDashboard(params['id']);
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  loadDashboard(id: string): void {
    this.loading = true;
    this.error = null;

    this.dashboardService.getDashboard(id)
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (dashboard) => {
          this.dashboard = dashboard;
          this.filters = dashboard.filters.map(f => ({
            id: f.id,
            value: null,
            filter: f
          }));
          this.loadCharts();
        },
        error: (error) => {
          this.error = 'Failed to load dashboard';
          this.loading = false;
        }
      });
  }

  loadCharts(): void {
    if (!this.dashboard) return;

    this.chartService.getChartsByDashboard(this.dashboard._id)
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (charts) => {
          this.charts = charts;
          this.loading = false;
        },
        error: (error) => {
          this.error = 'Failed to load charts';
          this.loading = false;
        }
      });
  }

  onFilterChange(filterId: string, value: any): void {
    const filter = this.filters.find(f => f.id === filterId);
    if (filter) {
      filter.value = value;
      this.updateCharts();
    }
  }

  updateCharts(): void {
    const activeFilters = this.filters.filter(f => f.value !== null);
    
    this.charts.forEach(chart => {
      this.chartService.updateChartWithFilters(chart._id, activeFilters)
        .pipe(takeUntil(this.destroy$))
        .subscribe({
          next: (updatedChart) => {
            const index = this.charts.findIndex(c => c._id === chart._id);
            if (index !== -1) {
              this.charts[index] = updatedChart;
            }
          }
        });
    });
  }

  onChartResize(chartId: string, size: { width: number; height: number }): void {
    this.chartService.resizeChart(chartId, size)
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: (updatedChart) => {
          const index = this.charts.findIndex(c => c._id === chartId);
          if (index !== -1) {
            this.charts[index] = updatedChart;
          }
        }
      });
  }
}
```

### Chart Component
```typescript
// components/chart/chart.component.ts
import { 
  Component, Input, OnInit, OnDestroy, 
  ElementRef, ViewChild, OnChanges, SimpleChanges 
} from '@angular/core';
import * as d3 from 'd3';

import { Chart, ChartData } from '../../models';

@Component({
  selector: 'app-chart',
  templateUrl: './chart.component.html',
  styleUrls: ['./chart.component.scss']
})
export class ChartComponent implements OnInit, OnChanges, OnDestroy {
  @Input() chart!: Chart;
  @Input() data: ChartData[] = [];
  @Input() filters: any[] = [];
  
  @ViewChild('chartContainer', { static: true }) 
  chartContainer!: ElementRef;

  private svg: d3.Selection<SVGSVGElement, unknown, null, undefined> | null = null;
  private margin = { top: 20, right: 20, bottom: 40, left: 40 };

  ngOnInit(): void {
    this.createChart();
  }

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['data'] || changes['filters']) {
      this.updateChart();
    }
  }

  ngOnDestroy(): void {
    if (this.svg) {
      this.svg.remove();
    }
  }

  createChart(): void {
    if (!this.chart || !this.data.length) return;

    const container = this.chartContainer.nativeElement;
    const width = container.clientWidth - this.margin.left - this.margin.right;
    const height = container.clientHeight - this.margin.top - this.margin.bottom;

    // Clear existing chart
    d3.select(container).selectAll('*').remove();

    // Create SVG
    this.svg = d3.select(container)
      .append('svg')
      .attr('width', width + this.margin.left + this.margin.right)
      .attr('height', height + this.margin.top + this.margin.bottom);

    const g = this.svg.append('g')
      .attr('transform', `translate(${this.margin.left},${this.margin.top})`);

    // Render based on chart type
    switch (this.chart.chart_type) {
      case 'bar':
        this.renderBarChart(g, width, height);
        break;
      case 'line':
        this.renderLineChart(g, width, height);
        break;
      case 'pie':
        this.renderPieChart(g, width, height);
        break;
      case 'scatter':
        this.renderScatterChart(g, width, height);
        break;
      default:
        this.renderBarChart(g, width, height);
    }
  }

  renderBarChart(g: d3.Selection<SVGGElement, unknown, null, undefined>, width: number, height: number): void {
    const xField = this.chart.configuration.x_axis.field;
    const yField = this.chart.configuration.y_axis.field;

    // Create scales
    const xScale = d3.scaleBand()
      .domain(this.data.map(d => d[xField]))
      .range([0, width])
      .padding(0.1);

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(this.data, d => d[yField]) || 0])
      .range([height, 0]);

    // Create axes
    g.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(xScale))
      .selectAll('text')
      .attr('transform', 'rotate(-45)')
      .style('text-anchor', 'end');

    g.append('g')
      .call(d3.axisLeft(yScale));

    // Create bars
    g.selectAll('.bar')
      .data(this.data)
      .enter()
      .append('rect')
      .attr('class', 'bar')
      .attr('x', d => xScale(d[xField]) || 0)
      .attr('width', xScale.bandwidth())
      .attr('y', d => yScale(d[yField]))
      .attr('height', d => height - yScale(d[yField]))
      .attr('fill', this.chart.configuration.styling.colors[0] || '#3b82f6');

    // Add labels
    g.append('text')
      .attr('x', width / 2)
      .attr('y', -10)
      .attr('text-anchor', 'middle')
      .style('font-size', '14px')
      .style('font-weight', 'bold')
      .text(this.chart.name);
  }

  renderLineChart(g: d3.Selection<SVGGElement, unknown, null, undefined>, width: number, height: number): void {
    const xField = this.chart.configuration.x_axis.field;
    const yField = this.chart.configuration.y_axis.field;

    // Parse dates if x-axis is time-based
    const parseTime = d3.timeParse('%Y-%m-%d');
    const processedData = this.data.map(d => ({
      ...d,
      [xField]: this.chart.configuration.x_axis.type === 'time' 
        ? parseTime(d[xField]) 
        : d[xField]
    }));

    // Create scales
    const xScale = this.chart.configuration.x_axis.type === 'time'
      ? d3.scaleTime()
          .domain(d3.extent(processedData, d => d[xField]) as [Date, Date])
          .range([0, width])
      : d3.scaleLinear()
          .domain(d3.extent(processedData, d => d[xField]) as [number, number])
          .range([0, width]);

    const yScale = d3.scaleLinear()
      .domain(d3.extent(processedData, d => d[yField]) as [number, number])
      .range([height, 0]);

    // Create line generator
    const line = d3.line<any>()
      .x(d => xScale(d[xField]))
      .y(d => yScale(d[yField]))
      .curve(d3.curveMonotoneX);

    // Create axes
    g.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(xScale));

    g.append('g')
      .call(d3.axisLeft(yScale));

    // Create line
    g.append('path')
      .datum(processedData)
      .attr('fill', 'none')
      .attr('stroke', this.chart.configuration.styling.colors[0] || '#3b82f6')
      .attr('stroke-width', 2)
      .attr('d', line);

    // Add dots
    g.selectAll('.dot')
      .data(processedData)
      .enter()
      .append('circle')
      .attr('class', 'dot')
      .attr('cx', d => xScale(d[xField]))
      .attr('cy', d => yScale(d[yField]))
      .attr('r', 4)
      .attr('fill', this.chart.configuration.styling.colors[0] || '#3b82f6');
  }

  renderPieChart(g: d3.Selection<SVGGElement, unknown, null, undefined>, width: number, height: number): void {
    const valueField = this.chart.configuration.y_axis.field;
    const labelField = this.chart.configuration.x_axis.field;

    // Create pie generator
    const pie = d3.pie<any>()
      .value(d => d[valueField])
      .sort(null);

    // Create arc generator
    const arc = d3.arc<any>()
      .innerRadius(0)
      .outerRadius(Math.min(width, height) / 2 - 10);

    // Create label arc
    const labelArc = d3.arc<any>()
      .innerRadius(Math.min(width, height) / 2 - 10)
      .outerRadius(Math.min(width, height) / 2);

    // Process data
    const pieData = pie(this.data);

    // Create arcs
    const arcs = g.selectAll('.arc')
      .data(pieData)
      .enter()
      .append('g')
      .attr('class', 'arc')
      .attr('transform', `translate(${width / 2}, ${height / 2})`);

    // Add paths
    arcs.append('path')
      .attr('d', arc)
      .attr('fill', (d, i) => this.chart.configuration.styling.colors[i % this.chart.configuration.styling.colors.length]);

    // Add labels
    arcs.append('text')
      .attr('transform', d => `translate(${labelArc.centroid(d)})`)
      .attr('text-anchor', 'middle')
      .style('font-size', '12px')
      .text(d => d.data[labelField]);
  }

  renderScatterChart(g: d3.Selection<SVGGElement, unknown, null, undefined>, width: number, height: number): void {
    const xField = this.chart.configuration.x_axis.field;
    const yField = this.chart.configuration.y_axis.field;
    const colorField = this.chart.configuration.color_field;
    const sizeField = this.chart.configuration.size_field;

    // Create scales
    const xScale = d3.scaleLinear()
      .domain(d3.extent(this.data, d => d[xField]) as [number, number])
      .range([0, width]);

    const yScale = d3.scaleLinear()
      .domain(d3.extent(this.data, d => d[yField]) as [number, number])
      .range([height, 0]);

    const colorScale = d3.scaleOrdinal()
      .domain([...new Set(this.data.map(d => d[colorField]))])
      .range(this.chart.configuration.styling.colors);

    const sizeScale = d3.scaleLinear()
      .domain(d3.extent(this.data, d => d[sizeField]) as [number, number])
      .range([4, 20]);

    // Create axes
    g.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(xScale));

    g.append('g')
      .call(d3.axisLeft(yScale));

    // Create dots
    g.selectAll('.dot')
      .data(this.data)
      .enter()
      .append('circle')
      .attr('class', 'dot')
      .attr('cx', d => xScale(d[xField]))
      .attr('cy', d => yScale(d[yField]))
      .attr('r', d => sizeField ? sizeScale(d[sizeField]) : 6)
      .attr('fill', d => colorField ? colorScale(d[colorField]) : this.chart.configuration.styling.colors[0])
      .attr('opacity', 0.7);

    // Add labels
    g.append('text')
      .attr('x', width / 2)
      .attr('y', -10)
      .attr('text-anchor', 'middle')
      .style('font-size', '14px')
      .style('font-weight', 'bold')
      .text(this.chart.name);
  }

  updateChart(): void {
    this.createChart();
  }
}
```

### Data Upload Component
```typescript
// components/data-upload/data-upload.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Router } from '@angular/router';

import { DataSourceService } from '../../services/data-source.service';
import { DatasetService } from '../../services/dataset.service';

@Component({
  selector: 'app-data-upload',
  templateUrl: './data-upload.component.html',
  styleUrls: ['./data-upload.component.scss']
})
export class DataUploadComponent implements OnInit {
  uploadForm: FormGroup;
  file: File | null = null;
  previewData: any[] = [];
  loading = false;
  error: string | null = null;

  constructor(
    private fb: FormBuilder,
    private router: Router,
    private dataSourceService: DataSourceService,
    private datasetService: DatasetService
  ) {
    this.uploadForm = this.fb.group({
      name: ['', Validators.required],
      description: [''],
      isPublic: [false],
      tags: ['']
    });
  }

  ngOnInit(): void {}

  onFileSelected(event: any): void {
    const file = event.target.files[0];
    if (file) {
      this.file = file;
      this.previewFile();
    }
  }

  previewFile(): void {
    if (!this.file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      const text = e.target?.result as string;
      this.parseCSV(text);
    };
    reader.readAsText(this.file);
  }

  parseCSV(text: string): void {
    const lines = text.split('\n');
    const headers = lines[0].split(',').map(h => h.trim());
    
    this.previewData = lines.slice(1, 6).map(line => {
      const values = line.split(',');
      const row: any = {};
      headers.forEach((header, index) => {
        row[header] = values[index]?.trim() || '';
      });
      return row;
    });
  }

  onSubmit(): void {
    if (!this.uploadForm.valid || !this.file) return;

    this.loading = true;
    this.error = null;

    const formData = new FormData();
    formData.append('file', this.file);
    formData.append('name', this.uploadForm.value.name);
    formData.append('description', this.uploadForm.value.description);
    formData.append('isPublic', this.uploadForm.value.isPublic);
    formData.append('tags', this.uploadForm.value.tags);

    this.datasetService.uploadDataset(formData)
      .subscribe({
        next: (dataset) => {
          this.router.navigate(['/datasets', dataset._id]);
        },
        error: (error) => {
          this.error = 'Failed to upload dataset';
          this.loading = false;
        }
      });
  }

  detectFieldTypes(): any {
    if (this.previewData.length === 0) return {};

    const headers = Object.keys(this.previewData[0]);
    const fieldTypes: any = {};

    headers.forEach(header => {
      const values = this.previewData.map(row => row[header]).filter(v => v !== '');
      
      if (values.every(v => !isNaN(Number(v)))) {
        fieldTypes[header] = 'number';
      } else if (values.every(v => !isNaN(Date.parse(v)))) {
        fieldTypes[header] = 'date';
      } else if (values.every(v => v === 'true' || v === 'false')) {
        fieldTypes[header] = 'boolean';
      } else {
        fieldTypes[header] = 'string';
      }
    });

    return fieldTypes;
  }
}
```

---

## 🛠️ Backend API (Node.js + Express)

### Chart Router
```javascript
// routes/charts.js
const express = require('express');
const router = express.Router();
const Chart = require('../models/Chart');
const Dataset = require('../models/Dataset');
const auth = require('../middleware/auth');

// Get all charts for a dashboard
router.get('/dashboard/:dashboardId', auth, async (req, res) => {
  try {
    const charts = await Chart.find({ 
      dashboard_id: req.params.dashboardId,
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    }).populate('dataset_id');

    res.json(charts);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch charts' });
  }
});

// Create a new chart
router.post('/', auth, async (req, res) => {
  try {
    const {
      name,
      description,
      dataset_id,
      chart_type,
      configuration,
      dashboard_id,
      position
    } = req.body;

    // Verify dataset access
    const dataset = await Dataset.findOne({
      _id: dataset_id,
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    });

    if (!dataset) {
      return res.status(403).json({ error: 'Access denied to dataset' });
    }

    const chart = new Chart({
      name,
      description,
      dataset_id,
      chart_type,
      configuration,
      dashboard_id,
      position,
      owner_id: req.user.id
    });

    await chart.save();
    await chart.populate('dataset_id');

    res.status(201).json(chart);
  } catch (error) {
    res.status(500).json({ error: 'Failed to create chart' });
  }
});

// Update chart with filters
router.post('/:id/update', auth, async (req, res) => {
  try {
    const chart = await Chart.findOne({
      _id: req.params.id,
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    }).populate('dataset_id');

    if (!chart) {
      return res.status(404).json({ error: 'Chart not found' });
    }

    // Apply filters to dataset
    const filteredData = await applyFilters(chart.dataset_id.data, req.body.filters);
    
    // Update chart data
    chart.data = filteredData;
    await chart.save();

    res.json(chart);
  } catch (error) {
    res.status(500).json({ error: 'Failed to update chart' });
  }
});

// Resize chart
router.post('/:id/resize', auth, async (req, res) => {
  try {
    const { width, height } = req.body;

    const chart = await Chart.findOneAndUpdate(
      { 
        _id: req.params.id,
        $or: [
          { owner_id: req.user.id },
          { is_public: true }
        ]
      },
      { 
        'configuration.styling.width': width,
        'configuration.styling.height': height
      },
      { new: true }
    );

    if (!chart) {
      return res.status(404).json({ error: 'Chart not found' });
    }

    res.json(chart);
  } catch (error) {
    res.status(500).json({ error: 'Failed to resize chart' });
  }
});

// Helper function to apply filters
async function applyFilters(data, filters) {
  if (!filters || filters.length === 0) {
    return data;
  }

  return data.filter(row => {
    return filters.every(filter => {
      const { field, operator, value } = filter.filter;
      const rowValue = row[field];

      switch (operator) {
        case 'eq':
          return rowValue === value;
        case 'gt':
          return rowValue > value;
        case 'lt':
          return rowValue < value;
        case 'contains':
          return String(rowValue).toLowerCase().includes(String(value).toLowerCase());
        case 'in':
          return Array.isArray(value) && value.includes(rowValue);
        default:
          return true;
      }
    });
  });
}

module.exports = router;
```

### Dataset Router
```javascript
// routes/datasets.js
const express = require('express');
const multer = require('multer');
const csv = require('csv-parser');
const fs = require('fs');
const path = require('path');
const router = express.Router();
const Dataset = require('../models/Dataset');
const auth = require('../middleware/auth');

const upload = multer({ 
  dest: 'uploads/',
  limits: { fileSize: 10 * 1024 * 1024 } // 10MB limit
});

// Get all datasets
router.get('/', auth, async (req, res) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;

    const datasets = await Dataset.find({
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    })
    .sort({ created_at: -1 })
    .skip(skip)
    .limit(limit);

    const total = await Dataset.countDocuments({
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    });

    res.json({
      datasets,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch datasets' });
  }
});

// Upload new dataset
router.post('/upload', auth, upload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }

    const { name, description, isPublic, tags } = req.body;
    const data = [];

    // Parse CSV file
    await new Promise((resolve, reject) => {
      fs.createReadStream(req.file.path)
        .pipe(csv())
        .on('data', (row) => data.push(row))
        .on('end', resolve)
        .on('error', reject);
    });

    // Clean up uploaded file
    fs.unlinkSync(req.file.path);

    // Create sample data (first 5 records)
    const sampleData = data.slice(0, 5);

    // Get column information
    const columns = data.length > 0 ? Object.keys(data[0]) : [];

    // Create dataset
    const dataset = new Dataset({
      name,
      description,
      data_source_id: null, // Manual upload
      data,
      metadata: {
        total_records: data.length,
        last_updated: new Date(),
        file_size: req.file.size,
        columns,
        sample_data: sampleData
      },
      owner_id: req.user.id,
      tags: tags ? tags.split(',').map(t => t.trim()) : [],
      is_public: isPublic === 'true'
    });

    await dataset.save();

    res.status(201).json(dataset);
  } catch (error) {
    res.status(500).json({ error: 'Failed to upload dataset' });
  }
});

// Get dataset by ID
router.get('/:id', auth, async (req, res) => {
  try {
    const dataset = await Dataset.findOne({
      _id: req.params.id,
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    });

    if (!dataset) {
      return res.status(404).json({ error: 'Dataset not found' });
    }

    res.json(dataset);
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch dataset' });
  }
});

// Get dataset preview (first 100 records)
router.get('/:id/preview', auth, async (req, res) => {
  try {
    const dataset = await Dataset.findOne({
      _id: req.params.id,
      $or: [
        { owner_id: req.user.id },
        { is_public: true }
      ]
    }).select('data metadata');

    if (!dataset) {
      return res.status(404).json({ error: 'Dataset not found' });
    }

    const previewData = dataset.data.slice(0, 100);

    res.json({
      data: previewData,
      metadata: dataset.metadata
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch dataset preview' });
  }
});

module.exports = router;
```

---

## 🧪 Testing Requirements

### Frontend Testing (Jasmine + Karma)
```typescript
// chart.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ChartComponent } from './chart.component';

describe('ChartComponent', () => {
  let component: ChartComponent;
  let fixture: ComponentFixture<ChartComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ChartComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(ChartComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should render bar chart with correct data', () => {
    component.chart = {
      _id: '1',
      name: 'Test Chart',
      chart_type: 'bar',
      configuration: {
        x_axis: { field: 'category', label: 'Category' },
        y_axis: { field: 'value', label: 'Value' },
        styling: { colors: ['#3b82f6'] }
      }
    } as any;

    component.data = [
      { category: 'A', value: 10 },
      { category: 'B', value: 20 },
      { category: 'C', value: 15 }
    ];

    fixture.detectChanges();
    
    const svg = fixture.nativeElement.querySelector('svg');
    expect(svg).toBeTruthy();
    expect(svg.querySelectorAll('rect.bar').length).toBe(3);
  });

  it('should update chart when data changes', () => {
    spyOn(component, 'updateChart');
    
    component.data = [{ test: 'data' }];
    component.ngOnChanges({ data: { currentValue: component.data } } as any);
    
    expect(component.updateChart).toHaveBeenCalled();
  });
});
```

### Backend Testing (Jest)
```javascript
// tests/charts.test.js
const request = require('supertest');
const express = require('express');
const chartRoutes = require('../routes/charts');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());
app.use('/charts', chartRoutes);

describe('Chart Routes', () => {
  beforeEach(async () => {
    // Setup test database
  });

  afterEach(async () => {
    // Cleanup test database
  });

  test('GET /charts/dashboard/:dashboardId should return charts', async () => {
    const response = await request(app)
      .get('/charts/dashboard/test-dashboard-id')
      .set('Authorization', 'Bearer test-token');

    expect(response.status).toBe(200);
    expect(Array.isArray(response.body)).toBe(true);
  });

  test('POST /charts should create new chart', async () => {
    const chartData = {
      name: 'Test Chart',
      description: 'Test description',
      dataset_id: 'test-dataset-id',
      chart_type: 'bar',
      configuration: {
        x_axis: { field: 'category' },
        y_axis: { field: 'value' },
        styling: { colors: ['#3b82f6'] }
      },
      dashboard_id: 'test-dashboard-id',
      position: { x: 0, y: 0, width: 400, height: 300 }
    };

    const response = await request(app)
      .post('/charts')
      .set('Authorization', 'Bearer test-token')
      .send(chartData);

    expect(response.status).toBe(201);
    expect(response.body.name).toBe('Test Chart');
    expect(response.body.chart_type).toBe('bar');
  });

  test('POST /charts/:id/update should apply filters', async () => {
    const filters = [{
      id: 'filter1',
      filter: {
        field: 'category',
        operator: 'eq',
        value: 'A'
      }
    }];

    const response = await request(app)
      .post('/charts/test-chart-id/update')
      .set('Authorization', 'Bearer test-token')
      .send({ filters });

    expect(response.status).toBe(200);
    expect(response.body.data).toBeDefined();
  });
});
```

---

## 🚀 Deployment Requirements

### Angular Deployment (Firebase/Vercel)
```json
// angular.json
{
  "projects": {
    "data-viz-platform": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/data-viz-platform",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": "src/polyfills.ts",
            "tsConfig": "tsconfig.app.json",
            "assets": [
              "src/favicon.ico",
              "src/assets"
            ],
            "styles": [
              "src/styles.scss"
            ],
            "scripts": []
          }
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "options": {
            "browserTarget": "data-viz-platform:build"
          }
        }
      }
    }
  }
}
```

### Node.js Deployment (Railway/Heroku)
```javascript
// server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');

const authRoutes = require('./routes/auth');
const chartRoutes = require('./routes/charts');
const datasetRoutes = require('./routes/datasets');
const dashboardRoutes = require('./routes/dashboards');

const app = express();

// Middleware
app.use(helmet());
app.use(compression());
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:4200',
  credentials: true
}));
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/charts', chartRoutes);
app.use('/api/datasets', datasetRoutes);
app.use('/api/dashboards', dashboardRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// Database connection
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/data-viz', {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => {
  console.log('Connected to MongoDB');
}).catch((error) => {
  console.error('MongoDB connection error:', error);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

---

## 📚 Learning Resources

### Essential Reading
- [Angular Documentation](https://angular.io/docs) - Enterprise web framework
- [D3.js Documentation](https://d3js.org/getting-started) - Data visualization library
- [Node.js Documentation](https://nodejs.org/docs/) - JavaScript runtime
- [MongoDB Documentation](https://docs.mongodb.com/) - NoSQL database

### Udemy Courses (Focus Sections)
- **Angular Complete Course** - Sections 4-9 (Advanced components, Routing, State management)
- **D3.js Data Visualization** - Sections 2-6 (Charts, Interactive graphics, Animations)
- **Node.js Complete Course** - Sections 5-8 (Express, MongoDB, Authentication)

### Practice Projects
- Build a simple data visualization dashboard
- Create an interactive chart library
- Implement a data analysis tool

---

## ✅ Success Criteria

### Week 7 Goals
- [ ] Set up Angular frontend with TypeScript
- [ ] Set up Node.js backend with Express and MongoDB
- [ ] Implement data upload and processing
- [ ] Create basic chart components with D3.js
- [ ] Build dashboard layout and navigation

### Week 8 Goals
- [ ] Implement advanced chart types and interactions
- [ ] Add filtering and data manipulation features
- [ ] Create responsive design and mobile support
- [ ] Add comprehensive testing
- [ ] Deploy to production
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional data visualization platform
- [ ] Multiple chart types with D3.js
- [ ] Interactive dashboard with filtering
- [ ] Data upload and management system
- [ ] Responsive Angular frontend
- [ ] Node.js backend with MongoDB
- [ ] 80%+ test coverage on both frontend and backend
- [ ] Deployed applications with live demos

---

## 🔗 Related Projects
- **Previous Project**: [Project 3: Project Management Tool](../Full_Stack/Project_3_Project_Management_Tool.md)
- **Training Plan**: [Full Stack Intensive Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.md)
- **Main README**: [Repository Overview](../../README.md)
