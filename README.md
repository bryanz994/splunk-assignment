# splunk-assignment

## Questions
## 1.In the Search app installed on your Splunk Enterprise version, find the top 5 viewed products referred by a domain.
```
sourcetype=access_* AND action=view
| top limit=5 productName by referer_domain
| stats list(productName) as productName, list(count) as count by referer_domain
```
![image](https://user-images.githubusercontent.com/19281954/174009553-4b92a8e9-ed2d-4533-bd8a-6823f0bab75f.png)

## 2.Find the best selling product and the total number of items sold by country.
```
sourcetype=access_* | iplocation clientip 
| stats count by productName Country 
| eventstats max(count) AS maximum, sum(count) AS Total by Country
| where count=maximum
| stats list(productName) AS "Best Seller", first(maximum) AS "Sold", first(Total) AS "Sold Total" by Country
```
![image](https://user-images.githubusercontent.com/19281954/174009489-6aa4a0d1-a217-4f6e-8f56-46a96827cd92.png)

## 3.Plot a trellis chart showing the average time spent on the Buttercup Games website for each user session by browser (Bonus Point).
```
sourcetype=access_* referer_domain="http://www.buttercupgames.com" 
| transaction JSESSIONID
| eval browser = case(match(useragent,"Firefox"),"Firefox", 
match(useragent,"Chrome"),"Chrome",
match(useragent,"Safari"),"Safari",
NOT match(useragent, "(Chrome|Safari|Firefox)"), "Others"
) 
| timechart span=1hr avg(duration) by browser
```
![image](https://user-images.githubusercontent.com/19281954/174009726-bbe8b2b0-14ed-4665-8792-f3226a5b02b4.png)

## 4.Create a dashboard with a base search and 3 - 5 panels of your choice with metrics that will portray ButterCup Games company performance

```
{
	"dataSources": {
		"ds_search_1_new_new_new_new_new_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com | transaction JSESSIONID | timechart span=1h sum(duration) as session_duration",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1_new_new_new_new_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com \r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"addtocart\"\r\n| timechart span=1h sum(count) as AddToCart\r\n| appendcols\r\n    [search sourcetype=access_* referer_domain=http://www.buttercupgames.com success\r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"purchase\" \r\n| timechart span=1h sum(count) as Purchased ]\r\n| eval Abandoned=AddToCart-Purchased\r\n| table _time, AddToCart, Purchased, Abandoned",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1_new_new_new_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com \r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"addtocart\"\r\n| timechart span=1h sum(count) as AddToCart\r\n| appendcols\r\n    [search sourcetype=access_* referer_domain=http://www.buttercupgames.com success\r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"purchase\" \r\n| timechart span=1h sum(count) as Purchased ]\r\n| eval Abandoned=AddToCart-Purchased\r\n| table _time, Abandoned",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1_new_new_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com success\n| stats count by action, _time\n| table _time, count, action\n| where action = \"purchase\"\n| timechart span=1h sum(count) as Purchased",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1_new_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com \r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"addtocart\"\r\n| timechart span=1h sum(count) as AddToCart",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1_new": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com \r\n| stats count by action, _time\r\n| table _time, count, action\r\n| where action = \"addtocart\"\r\n| timechart span=1h sum(count) as AddToCart",
				"queryParameters": {
					"earliest": "0"
				}
			}
		},
		"ds_search_1": {
			"type": "ds.search",
			"options": {
				"query": "sourcetype=access_* referer_domain=http://www.buttercupgames.com GET action=addtocart \r\n| timechart span=1h dc(JSESSIONID) AS Cart\r\n| appendcols \r\n    [search sourcetype=access_* referer_domain=http://www.buttercupgames.com \"POST /cart/success\" action=purchase \r\n    | timechart span=1h dc(JSESSIONID) AS Purchased] \r\n| eval Abandoned=if(Cart<Purchased,null,Cart-Purchased)\r\n| table _time, Cart, Purchased, Abandoned",
				"queryParameters": {
					"earliest": "0"
				}
			}
		}
	},
	"visualizations": {
		"viz_single_1_new_new_new": {
			"type": "splunk.singlevalue",
			"options": {
				"colorMode": "none",
				"drilldown": "none",
				"numberPrecision": 0,
				"sparklineDisplay": "below",
				"trendDisplay": "percent",
				"trellis.enabled": 0,
				"trellis.scales.shared": 1,
				"trellis.size": "medium",
				"unitPosition": "after",
				"shouldUseThousandSeparators": true,
				"trendColor": "> trendValue | rangeValue(trendColorEditorConfig)",
				"majorColor": "#f8be34",
				"sparklineStrokeColor": "#f8be34"
			},
			"dataSources": {
				"primary": "ds_search_1_new_new_new_new_new_new"
			},
			"context": {
				"trendColorEditorConfig": [
					{
						"to": 0,
						"value": "#9E2520"
					},
					{
						"from": 0,
						"value": "#1C6B2D"
					}
				]
			},
			"title": "In the last 1 hour"
		},
		"viz_chart_1_new": {
			"type": "viz.column",
			"options": {
				"axisLabelsX.majorLabelStyle.overflowMode": "ellipsisNone",
				"axisLabelsX.majorLabelStyle.rotation": 0,
				"axisTitleX.visibility": "visible",
				"axisTitleY.visibility": "visible",
				"axisTitleY2.visibility": "visible",
				"axisX.abbreviation": "none",
				"axisX.scale": "linear",
				"axisY.abbreviation": "none",
				"axisY.scale": "linear",
				"axisY2.abbreviation": "none",
				"axisY2.enabled": 0,
				"axisY2.scale": "inherit",
				"chart.bubbleMaximumSize": 50,
				"chart.bubbleMinimumSize": 10,
				"chart.bubbleSizeBy": "area",
				"chart.nullValueMode": "gaps",
				"chart.showDataLabels": "none",
				"chart.sliceCollapsingThreshold": 0.01,
				"chart.style": "shiny",
				"drilldown": "none",
				"layout.splitSeries": 0,
				"layout.splitSeries.allowIndependentYRanges": 0,
				"legend.labelStyle.overflowMode": "ellipsisMiddle",
				"legend.mode": "standard",
				"legend.placement": "right",
				"lineWidth": 2,
				"trellis.enabled": 0,
				"trellis.scales.shared": 1,
				"trellis.size": "medium",
				"chart.stackMode": "stacked"
			},
			"dataSources": {
				"primary": "ds_search_1_new_new_new_new_new"
			}
		},
		"viz_single_1_new_new": {
			"type": "splunk.singlevalue",
			"options": {
				"colorMode": "none",
				"drilldown": "none",
				"numberPrecision": 0,
				"sparklineDisplay": "below",
				"trendDisplay": "percent",
				"trellis.enabled": 0,
				"trellis.scales.shared": 1,
				"trellis.size": "medium",
				"unitPosition": "after",
				"shouldUseThousandSeparators": true,
				"trendColor": "> trendValue | rangeValue(trendColorEditorConfig)",
				"sparklineStrokeColor": "#D41F1F"
			},
			"dataSources": {
				"primary": "ds_search_1_new_new_new_new"
			},
			"title": "In the last 1 hour",
			"context": {
				"trendColorEditorConfig": [
					{
						"to": 0,
						"value": "#9E2520"
					},
					{
						"from": 0,
						"value": "#1C6B2D"
					}
				]
			}
		},
		"viz_single_1_new": {
			"type": "splunk.singlevalue",
			"options": {
				"colorMode": "none",
				"drilldown": "none",
				"numberPrecision": 0,
				"sparklineDisplay": "below",
				"trendDisplay": "percent",
				"trellis.enabled": 0,
				"trellis.scales.shared": 1,
				"trellis.size": "medium",
				"unitPosition": "after",
				"shouldUseThousandSeparators": true,
				"trendColor": "> trendValue | rangeValue(trendColorEditorConfig)",
				"sparklineStrokeColor": "#D41F1F"
			},
			"dataSources": {
				"primary": "ds_search_1_new_new_new"
			},
			"title": "In the last 1 hour",
			"context": {
				"trendColorEditorConfig": [
					{
						"value": "#F98C83",
						"to": 0
					},
					{
						"value": "#55C169",
						"from": 0
					}
				]
			}
		},
		"viz_single_1": {
			"type": "splunk.singlevalue",
			"options": {
				"colorMode": "none",
				"drilldown": "none",
				"numberPrecision": 0,
				"sparklineDisplay": "below",
				"trendDisplay": "percent",
				"trellis.enabled": 0,
				"trellis.scales.shared": 1,
				"trellis.size": "medium",
				"unitPosition": "after",
				"shouldUseThousandSeparators": true,
				"trendColor": "> trendValue | rangeValue(trendColorEditorConfig)",
				"sparklineStrokeColor": "#D41F1F",
				"majorColor": "#ffffff"
			},
			"dataSources": {
				"primary": "ds_search_1_new"
			},
			"title": "In the last 1 hour",
			"context": {
				"trendColorEditorConfig": [
					{
						"value": "#F98C83",
						"to": 0
					},
					{
						"value": "#55C169",
						"from": 0
					}
				]
			}
		},
		"viz_SL3ZmenG": {
			"type": "viz.text",
			"options": {
				"content": "Add to cart"
			}
		},
		"viz_ZNMMDbfK": {
			"type": "viz.text",
			"options": {
				"content": "Checkouts"
			}
		},
		"viz_5MosjQbf": {
			"type": "viz.text",
			"options": {
				"content": "Abandoned Baskets"
			}
		},
		"viz_IdOPZ7aV": {
			"type": "viz.text",
			"options": {
				"content": "Cart Abandonment rate"
			}
		},
		"viz_8Jy3HckP": {
			"type": "viz.text",
			"options": {
				"content": "Session duration (s)"
			}
		}
	},
	"inputs": {
		"input_global_trp": {
			"type": "input.timerange",
			"options": {
				"token": "global_time",
				"defaultValue": "-24h@h,now"
			},
			"title": "Global Time Range"
		}
	},
	"layout": {
		"type": "absolute",
		"options": {
			"display": "auto-scale"
		},
		"structure": [
			{
				"item": "viz_single_1",
				"type": "block",
				"position": {
					"x": 0,
					"y": 50,
					"w": 250,
					"h": 170
				}
			},
			{
				"item": "viz_SL3ZmenG",
				"type": "block",
				"position": {
					"x": 10,
					"y": 0,
					"w": 190,
					"h": 50
				}
			},
			{
				"item": "viz_ZNMMDbfK",
				"type": "block",
				"position": {
					"x": 280,
					"y": 0,
					"w": 300,
					"h": 50
				}
			},
			{
				"item": "viz_5MosjQbf",
				"type": "block",
				"position": {
					"x": 550,
					"y": 0,
					"w": 300,
					"h": 50
				}
			},
			{
				"item": "viz_single_1_new",
				"type": "block",
				"position": {
					"x": 270,
					"y": 50,
					"w": 250,
					"h": 170
				}
			},
			{
				"item": "viz_single_1_new_new",
				"type": "block",
				"position": {
					"x": 540,
					"y": 50,
					"w": 250,
					"h": 170
				}
			},
			{
				"item": "viz_chart_1_new",
				"type": "block",
				"position": {
					"x": 0,
					"y": 280,
					"w": 1090,
					"h": 300
				}
			},
			{
				"item": "viz_IdOPZ7aV",
				"type": "block",
				"position": {
					"x": 0,
					"y": 230,
					"w": 300,
					"h": 50
				}
			},
			{
				"item": "viz_single_1_new_new_new",
				"type": "block",
				"position": {
					"x": 810,
					"y": 50,
					"w": 280,
					"h": 170
				}
			},
			{
				"item": "viz_8Jy3HckP",
				"type": "block",
				"position": {
					"x": 820,
					"y": 0,
					"w": 300,
					"h": 50
				}
			}
		],
		"globalInputs": [
			"input_global_trp"
		]
	},
	"title": "ButterCup",
	"description": "",
	"defaults": {
		"dataSources": {
			"ds.search": {
				"options": {
					"queryParameters": {
						"latest": "$global_time.latest$",
						"earliest": "$global_time.earliest$"
					}
				}
			}
		}
	}
}
```

