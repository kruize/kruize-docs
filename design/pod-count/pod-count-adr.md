# [Kruize | ROS] Include current replica count in recommendation API response

**Version:** 0.6<br>
**Date:** 09th March 2026<br>
**Status:** Under Review

## Document Log

| Revision | Date | Description | Author(s) |
| --- | --- | --- | --- |
| 0.1 | 09-03-2026 | Initial Outline | Bhaktavatsal Reddy |
| 0.2 | 10-03-2026 | Include feedback from design review meeting @ 09th March 2026 | Bhaktavatsal Reddy |
| 0.3 | 11-03-2026 | Added tasks for ROS and Cost Management | Bhaktavatsal Reddy |
| 0.4 | 16-03-2026 | Addressed feedback from ROS team | Bhaktavatsal Reddy |
| 0.5 | 17-03-2026 | Updated task to make changes to Kruize tests | Bhaktavatsal Reddy |
| 0.6 | 26-03-2026 | Update task to release breaking changes as part of new API | Bhaktavatsal Reddy |

## Review Log

| Date | Feedback | Reviewer |
| --- | --- | --- |
| 09-03-2026 | Include 'avg' in kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].current.replicas | Kusuma Chalasani |
| 09-03-2026 | Modify kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].config.replicas to hold number instead of JSONObject | Dinakar Guniguntala |
| 09-03-2026 | When HPA recommendations are generated in future, it will be maintained in separate config outside containers as they are specific to deployment | Bharath Appali |
| 09-03-2026 | Flip the structure to start from terms down to containers and recommendation and compare the size | Dinakar Guniguntala |
| 10-03-2026 | Include tasks related to ROS and discuss with ROS team ASAP | Dinakar Guniguntala |
| 13-03-2026 | change key kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].current.replicas to pod_count | Sagnik Dutta |
| 13-03-2026 | key 'replicas' is of different datatype. in different context. It might cause confusion | Kavita Gaikwad |
| 13-03-2026 | include replicas of last datapoint in kubernetes_objects[n].containers[n].recommendations.data.[ts].current | Sagnik Dutta |
| 17-03-2026 | Update kruize test as changes include breaking changes | Chandrakala |
| 25-03-2026 | Release breaking changes part of new version of API instead of existing | Sagnik Dutta |
| 25-03-2026 | kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].metrics_info can be renamed to something else | Sagnik Dutta |

## Approval Log

| Role | Name | Date | Comments |
| --- | --- | --- | --- |
| Technical Lead | Kusuma Chalasani | | |
| Chief Architect | Dinakar Guniguntala | | |
| Project Manager | Rashmi Badagandi | | |
| ROS Backend | Kavita Gaikwad | | |

## Problem Statement

Kruize provide recommendations for static values of resource requests and limits which are to set at pod specification. This could be either increase or decrease in value compared to their current configuration and has impact on cost. Redhat Console, where our recommendations are shown to customers has only CPU and Memory information. It doesn't contain no.of pods making it difficult for customer to assess the overall cost implication of our recommendations.

For reference, below figure shows how our recommendations are shown in Redhat console.
![UI](kruize_reco_in_ui.png)

## Context

Redhat console renders the information returned by Kruize API endpoint /updateRecommendations.

<details>
  <summary>API Response</summary>

  <!-- Add a blank line after the summary tag for best results -->
  
  ```json
  [
  {
    "cluster_name": "default",
    "experiment_type": "container",
    "kubernetes_objects": [
      {
        "type": "deployment",
        "name": "tfb-qrh-sample-0",
        "namespace": "default",
        "containers": [
          {
            "container_image_name": "kruize/tfb-qrh:1.13.2.F_et17",
            "container_name": "tfb-server",
            "recommendations": {
              "version": "1.0",
              "notifications": {
                "111000": {
                  "type": "info",
                  "message": "Recommendations Are Available",
                  "code": 111000
                }
              },
              "data": {
                "2025-11-17T10:31:51.000Z": {
                  "notifications": {
                    "111101": {
                      "type": "info",
                      "message": "Short Term Recommendations Available",
                      "code": 111101
                    },
                    "111103": {
                      "type": "info",
                      "message": "Long Term Recommendations Available",
                      "code": 111103
                    },
                    "111102": {
                      "type": "info",
                      "message": "Medium Term Recommendations Available",
                      "code": 111102
                    }
                  },
                  "monitoring_end_time": "2025-11-17T10:31:51.000Z",
                  "current": {
                    "limits": {
                      "memory": {
                        "amount": 5450498048,
                        "format": "bytes"
                      },
                      "cpu": {
                        "amount": 4.65,
                        "format": "cores"
                      }
                    },
                    "requests": {
                      "memory": {
                        "amount": 5450498048,
                        "format": "bytes"
                      },
                      "cpu": {
                        "amount": 4.65,
                        "format": "cores"
                      }
                    }
                  },
                  "recommendation_terms": {
                    "short_term": {
                      "duration_in_hours": 24.0,
                      "notifications": {
                        "112101": {
                          "type": "info",
                          "message": "Cost Recommendations Available",
                          "code": 112101
                        },
                        "112102": {
                          "type": "info",
                          "message": "Performance Recommendations Available",
                          "code": 112102
                        }
                      },
                      "monitoring_start_time": "2025-11-16T10:31:51.000Z",
                      "recommendation_engines": {
                        "cost": {
                          "pods_count": 2,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5303476224,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.011189027927476171,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5303476224,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.011189027927476171,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "1"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": -147021824,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.638810972072524,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": -147021824,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.638810972072524,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        },
                        "performance": {
                          "pods_count": 2,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5303476224,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.011189027927476171,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5303476224,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.011189027927476171,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "1"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": -147021824,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.638810972072524,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": -147021824,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.638810972072524,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        }
                      },
                      "plots": {
                        "datapoints": 4,
                        "plots_data": {
                          "2025-11-16T16:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00025783131690936326,
                              "q1": 0.006255391531935473,
                              "median": 0.00660694604704934,
                              "q3": 0.006957990934985269,
                              "max": 0.00826291905933281,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729501696,
                              "q1": 4418850816,
                              "median": 4418973696,
                              "q3": 4419039232,
                              "max": 4419301376,
                              "format": "bytes"
                            }
                          },
                          "2025-11-17T04:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002610067281416783,
                              "q1": 0.006380043822214618,
                              "median": 0.007248875719207945,
                              "q3": 0.009210413948556779,
                              "max": 0.011189027927476171,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729530368,
                              "q1": 4.41884672E+9,
                              "median": 4419047424,
                              "q3": 4.41905152E+9,
                              "max": 4419284992,
                              "format": "bytes"
                            }
                          },
                          "2025-11-16T22:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002612116488432934,
                              "q1": 0.006302020791036147,
                              "median": 0.006813888179417914,
                              "q3": 0.006963045986443549,
                              "max": 0.008390761613527961,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729526272,
                              "q1": 4419043328,
                              "median": 4.41909248E+9,
                              "q3": 4419137536,
                              "max": 4419301376,
                              "format": "bytes"
                            }
                          },
                          "2025-11-17T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002530158531450844,
                              "q1": 0.006630715207110741,
                              "median": 0.007181887330650302,
                              "q3": 0.007715433945082201,
                              "max": 0.009666150179108842,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729530368,
                              "q1": 4418998272,
                              "median": 4419043328,
                              "q3": 4.41905152E+9,
                              "max": 4.41956352E+9,
                              "format": "bytes"
                            }
                          }
                        }
                      }
                    },
                    "medium_term": {
                      "duration_in_hours": 168.0,
                      "notifications": {
                        "112101": {
                          "type": "info",
                          "message": "Cost Recommendations Available",
                          "code": 112101
                        },
                        "112102": {
                          "type": "info",
                          "message": "Performance Recommendations Available",
                          "code": 112102
                        }
                      },
                      "monitoring_start_time": "2025-11-10T10:31:51.000Z",
                      "recommendation_engines": {
                        "cost": {
                          "pods_count": 4,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.007816995421441963,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.007816995421441963,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "1"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.642183004578558,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.642183004578558,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        },
                        "performance": {
                          "pods_count": 4,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 14.100655972488788,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 14.100655972488788,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "15"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 9.450655972488788,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 9.450655972488788,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        }
                      },
                      "plots": {
                        "datapoints": 7,
                        "plots_data": {
                          "2025-11-15T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00025026570867423865,
                              "q1": 0.00642221401495096,
                              "median": 0.0067916434544654245,
                              "q3": 0.007544279109308003,
                              "max": 0.010853665646195745,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 727310336,
                              "q1": 4418945024,
                              "median": 4.41905152E+9,
                              "q3": 4419178496,
                              "max": 4.41954304E+9,
                              "format": "bytes"
                            }
                          },
                          "2025-11-13T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00025305864431669116,
                              "q1": 0.00664030893674273,
                              "median": 0.007294561987482653,
                              "q3": 0.00814570731216644,
                              "max": 0.010768459549283411,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729239552,
                              "q1": 4418940928,
                              "median": 4419043328,
                              "q3": 4419203072,
                              "max": 4419452928,
                              "format": "bytes"
                            }
                          },
                          "2025-11-14T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002291695607209723,
                              "q1": 0.006501464355720669,
                              "median": 0.007069191118230723,
                              "q3": 0.008020747322413943,
                              "max": 0.011284064665168604,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729235456,
                              "q1": 4418940928,
                              "median": 4419039232,
                              "q3": 4419104768,
                              "max": 4419567616,
                              "format": "bytes"
                            }
                          },
                          "2025-11-11T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0010112515293466582,
                              "q1": 6.117722531153521,
                              "median": 8.614557689975573,
                              "q3": 12.76056902913985,
                              "max": 28.04292016087969,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 262144.0,
                              "q1": 4423929856,
                              "median": 4824875008,
                              "q3": 4879089664,
                              "max": 4991946752,
                              "format": "bytes"
                            }
                          },
                          "2025-11-16T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00024414165808636304,
                              "q1": 0.006460396208816531,
                              "median": 0.006904225614116109,
                              "q3": 0.0077778344722605345,
                              "max": 0.012641922366769223,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 7.2943616E+8,
                              "q1": 4418957312,
                              "median": 4419043328,
                              "q3": 4419231744,
                              "max": 4419510272,
                              "format": "bytes"
                            }
                          },
                          "2025-11-17T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002530158531450844,
                              "q1": 0.006351025036021565,
                              "median": 0.006813888179417914,
                              "q3": 0.007669462689186835,
                              "max": 0.011189027927476171,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729501696,
                              "q1": 4418940928,
                              "median": 4419043328,
                              "q3": 4.41909248E+9,
                              "max": 4.41956352E+9,
                              "format": "bytes"
                            }
                          },
                          "2025-11-12T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0011372446015659858,
                              "q1": 0.007363955119015076,
                              "median": 0.00807116001470862,
                              "q3": 0.009177754892155731,
                              "max": 0.0228623477419697,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 771469312,
                              "q1": 4418994176,
                              "median": 4419055616,
                              "q3": 4419207168,
                              "max": 4.41946112E+9,
                              "format": "bytes"
                            }
                          }
                        }
                      }
                    },
                    "long_term": {
                      "duration_in_hours": 360.0,
                      "notifications": {
                        "112101": {
                          "type": "info",
                          "message": "Cost Recommendations Available",
                          "code": 112101
                        },
                        "112102": {
                          "type": "info",
                          "message": "Performance Recommendations Available",
                          "code": 112102
                        }
                      },
                      "monitoring_start_time": "2025-11-02T10:31:51.000Z",
                      "recommendation_engines": {
                        "cost": {
                          "pods_count": 5,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.009717069764110815,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 0.009717069764110815,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "1"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.640282930235889,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": -4.640282930235889,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        },
                        "performance": {
                          "pods_count": 5,
                          "confidence_level": 0.0,
                          "config": {
                            "requests": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 14.338410549492885,
                                "format": "cores"
                              }
                            },
                            "limits": {
                              "memory": {
                                "amount": 5990336102.4,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 14.338410549492885,
                                "format": "cores"
                              }
                            },
                            "env": [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              },
                              {
                                "name": "QUARKUS_THREAD_POOL_CORE_THREADS",
                                "value": "15"
                              }
                            ]
                          },
                          "variation": {
                            "limits": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 9.688410549492884,
                                "format": "cores"
                              }
                            },
                            "requests": {
                              "memory": {
                                "amount": 539838054.3999996,
                                "format": "bytes"
                              },
                              "cpu": {
                                "amount": 9.688410549492884,
                                "format": "cores"
                              }
                            }
                          },
                          "notifications": {}
                        }
                      },
                      "plots": {
                        "datapoints": 15,
                        "plots_data": {
                          "2025-11-03T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002381760047787447,
                              "q1": 1.1243251327491783,
                              "median": 4.750352914730304,
                              "q3": 5.226372661634674,
                              "max": 5.795487708646885,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 3566563328,
                              "q1": 4331917312,
                              "median": 4390998016,
                              "q3": 4426838016,
                              "max": 4541538304,
                              "format": "bytes"
                            }
                          },
                          "2025-11-08T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0007372733259531948,
                              "q1": 0.003543077823924483,
                              "median": 0.0037434685583833285,
                              "q3": 5.5745121808460585,
                              "max": 11.238946351545973,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 3905552384,
                              "q1": 4.41278464E+9,
                              "median": 4412940288,
                              "q3": 4427194368,
                              "max": 4432257024,
                              "format": "bytes"
                            }
                          },
                          "2025-11-13T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00025305864431669116,
                              "q1": 0.00664030893674273,
                              "median": 0.007294561987482653,
                              "q3": 0.00814570731216644,
                              "max": 0.010768459549283411,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729239552,
                              "q1": 4418940928,
                              "median": 4419043328,
                              "q3": 4419203072,
                              "max": 4419452928,
                              "format": "bytes"
                            }
                          },
                          "2025-11-14T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002291695607209723,
                              "q1": 0.006501464355720669,
                              "median": 0.007069191118230723,
                              "q3": 0.008020747322413943,
                              "max": 0.011284064665168604,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729235456,
                              "q1": 4418940928,
                              "median": 4419039232,
                              "q3": 4419104768,
                              "max": 4419567616,
                              "format": "bytes"
                            }
                          },
                          "2025-11-04T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002685496300912027,
                              "q1": 0.0019189593852750673,
                              "median": 0.0020876918625935604,
                              "q3": 5.233634494574109,
                              "max": 16.125939425113422,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 3578200064,
                              "q1": 4.3227136E+9,
                              "median": 4.3230208E+9,
                              "q3": 4.3656192E+9,
                              "max": 4.50521088E+9,
                              "format": "bytes"
                            }
                          },
                          "2025-11-09T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.000727278146428902,
                              "q1": 0.0034979905824156724,
                              "median": 0.003585170351686009,
                              "q3": 0.0036921114309484745,
                              "max": 0.004092753614117215,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 3921117184,
                              "q1": 4412715008,
                              "median": 4412862464,
                              "q3": 4412940288,
                              "max": 4413202432,
                              "format": "bytes"
                            }
                          },
                          "2025-11-05T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002898730411586647,
                              "q1": 5.041469132812894,
                              "median": 5.9258460593112705,
                              "q3": 9.349926678275715,
                              "max": 16.343330362148166,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 789815296,
                              "q1": 4413022208,
                              "median": 4419211264,
                              "q3": 4431998976,
                              "max": 4465668096,
                              "format": "bytes"
                            }
                          },
                          "2025-11-10T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0007561210072779026,
                              "q1": 0.0035447534317902736,
                              "median": 0.003663422498800419,
                              "q3": 0.00394406345009722,
                              "max": 6.203321037887015,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 730136576,
                              "q1": 4.41278464E+9,
                              "median": 4412870656,
                              "q3": 4413120512,
                              "max": 4429955072,
                              "format": "bytes"
                            }
                          },
                          "2025-11-15T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00025026570867423865,
                              "q1": 0.00642221401495096,
                              "median": 0.0067916434544654245,
                              "q3": 0.007544279109308003,
                              "max": 0.010853665646195745,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 727310336,
                              "q1": 4418945024,
                              "median": 4.41905152E+9,
                              "q3": 4419178496,
                              "max": 4.41954304E+9,
                              "format": "bytes"
                            }
                          },
                          "2025-11-11T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0010112515293466582,
                              "q1": 6.117722531153521,
                              "median": 8.614557689975573,
                              "q3": 12.76056902913985,
                              "max": 28.04292016087969,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 262144.0,
                              "q1": 4423929856,
                              "median": 4824875008,
                              "q3": 4879089664,
                              "max": 4991946752,
                              "format": "bytes"
                            }
                          },
                          "2025-11-16T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.00024414165808636304,
                              "q1": 0.006460396208816531,
                              "median": 0.006904225614116109,
                              "q3": 0.0077778344722605345,
                              "max": 0.012641922366769223,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 7.2943616E+8,
                              "q1": 4418957312,
                              "median": 4419043328,
                              "q3": 4419231744,
                              "max": 4419510272,
                              "format": "bytes"
                            }
                          },
                          "2025-11-06T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0009060137518577036,
                              "q1": 6.484936379930901,
                              "median": 9.967065609697386,
                              "q3": 13.918576504201667,
                              "max": 14.77150274309032,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 1158873088,
                              "q1": 4414930944,
                              "median": 4423041024,
                              "q3": 4431446016,
                              "max": 4453842944,
                              "format": "bytes"
                            }
                          },
                          "2025-11-17T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0002530158531450844,
                              "q1": 0.006351025036021565,
                              "median": 0.006813888179417914,
                              "q3": 0.007669462689186835,
                              "max": 0.011189027927476171,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 729501696,
                              "q1": 4418940928,
                              "median": 4419043328,
                              "q3": 4.41909248E+9,
                              "max": 4.41956352E+9,
                              "format": "bytes"
                            }
                          },
                          "2025-11-07T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.000852395065027912,
                              "q1": 5.193707354037056,
                              "median": 6.297400503320733,
                              "q3": 7.761887644308369,
                              "max": 11.306369025482608,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 3.84731136E+9,
                              "q1": 4.43017216E+9,
                              "median": 4431454208,
                              "q3": 4433788928,
                              "max": 4447023104,
                              "format": "bytes"
                            }
                          },
                          "2025-11-12T10:31:51.000Z": {
                            "cpuUsage": {
                              "min": 0.0011372446015659858,
                              "q1": 0.007363955119015076,
                              "median": 0.00807116001470862,
                              "q3": 0.009177754892155731,
                              "max": 0.0228623477419697,
                              "format": "cores"
                            },
                            "memoryUsage": {
                              "min": 771469312,
                              "q1": 4418994176,
                              "median": 4419055616,
                              "q3": 4419207168,
                              "max": 4.41946112E+9,
                              "format": "bytes"
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        ]
      }
    ],
    "version": "v2.0",
    "experiment_name": "monitor_tfb_benchmark"
  }
]
```
  
</details>

Simplified version by excluding the repetitions is as below

```json
[
  {
    "cluster_name": "default",
    "experiment_name": "monitor_tfb_benchmark",
    "experiment_type": "container",
    "kubernetes_objects":
    [
      {
        "containers":
        [
          {
            "container_image_name": "kruize/tfb-qrh:1.13.2.F_et17",
            "container_name": "tfb-server",
            "recommendations":
            {
              "data":
              {
                "2025-11-17T10:31:51.000Z":
                {
                  "current":
                  {
                    "limits": {},
                    "requests": {}
                  },
                  "monitoring_end_time": "2025-11-17T10:31:51.000Z",
                  "recommendation_terms":
                  {
                    "short_term":
                    {
                      "duration_in_hours": 24.0,
                      "monitoring_start_time": "2025-11-16T10:31:51.000Z",
                      "recommendation_engines":
                      {
                        "cost":
                        {
                          "pods_count": 2,
                          "config":
                          {
                            "limits": {},
                            "requests": {},
                            "env":
                            [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              }
                            ]
                          },
                          "variation":
                          {
                            "limits": {},
                            "requests": {}
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        ],
        "name": "tfb-qrh-sample-0",
        "namespace": "default",
        "type": "deployment"
      }
    ],
    "version": "v2.0"
  }
]
```

## Design Considerations

### Metrics Datapoints

Each datapoint of metrics consist of aggregation values like min, max, avg and sum of metrics over a fixed time interval of all pods. By default, this interval is 15 mins. This means, 4 datapoints every 1 hr per metric. Minimum 2 datapoints are required for generating recommendations.

Datapoints of following metrics are available today both at workload level and namespace level.

1. cpu.requests
2. cpu.limits
3. cpu.usage
4. cpu.throttle
5. memory.requests
6. memory.limits
7. memory.usage
8. memory.rss

Apart from these, there are additional metrics are collected related to GPU recommendations.

### Recommendation Terms

Depending on no.of datapoints chosen for generating recommendations, there are broadly 3 terms defined.

- Short Term - based on datapoints of last 1 day (4x24 = 96 datapoints)
- Medium Term - based on datapoints of last 7 days (4x24x7 = 672 datapoints)
- Long Term - based on datapoints of last 15 days (4x24x15 = 1440 datapoints)

### Pod count

Pod count is not collected today. However, pod count is computed and returned in API response for each recommendation terms we support. pod count is computed from the sum and avg of metrics 'cpu.usage'. Value we return is the max(Math.floor(sum / avg)) for each term.

Resource requests and limits shown in current configuration is of the last 1 datapoint. Similarly, pod count of last 1 interval also can be shown.

### Recommendation for Pod count

Just like static values for resource requests and limits, we can also recommend static values for pod count to be used in deployment in future. And, this could be different based on the recommendation term.

### HPA & VPA recommendations

Currently, we neither collect the HPA and VPA configuration if applied for a workload nor generate HPA & VPA configuration for any workload. If we have to support these, API response structure is going to change a lot.

<details>
  <summary>Sample HPA configuration</summary>
  
  ```yaml
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: my-app-hpa
  spec:
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: my-app-deployment
    minReplicas: 1
    maxReplicas: 10
    metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
    behavior:
      scaleUp:
        # Allows rapid scale-out using the most aggressive policy (Max)
        # when multiple policies are defined.
        selectPolicy: Max
        policies:
        # Policy 1: Add 4 pods at a time, every 15 seconds.
        - type: Pods
          value: 4
          periodSeconds: 15
        # Policy 2: Add 100% of the current running pods every 15 seconds,
        # allowing for exponential growth.
        - type: Percent
          value: 100
          periodSeconds: 15
      scaleDown:
        # Use a stabilization window to prevent rapid scale-down due to
        # transient metric drops.
        stabilizationWindowSeconds: 300
        # Use the most conservative policy (Min) to ensure gradual reduction
        # of pods for stability.
        selectPolicy: Min
        policies:
        # Policy 1: Remove 1 pod at a time, every 60 seconds.
        - type: Pods
          value: 1
          periodSeconds: 60
        # Policy 2: Remove 10% of the current running pods every 60 seconds.
        - type: Percent
          value: 10
          periodSeconds: 60
  ```

</details>

<details>
  <summary>Sample VPA configuration</summary>
  
  ```yaml
  apiVersion: autoscaling.k8s.io/v1
  kind: VerticalPodAutoscaler
  metadata:
    name: app-vpa
  spec:
    targetRef:
      apiVersion: "apps/v1"
      kind: "Deployment"
      name: "target-deployment"
    updatePolicy:
      updateMode: "Auto" # Actively updates resources
    resourcePolicy:
      containerPolicies:
      - containerName: "*"
        minAllowed: {cpu: 50m, memory: 64Mi}
        maxAllowed: {cpu: 1, memory: 512Mi}

  ```
  
</details>

### Prometheus Metrics

There are different metrics available to get desired pod count & actual pod count for deployment and statefulset.

1. kube_statefulset_replicas (to get desired replica count of stateful app)
2. kube_deployment_spec_replicas (to get desired replica count of stateless app)
3. kube_pod_container_status_ready (to get pods which are in ready state)

Depending on decision to show desired or actual pods, we can add query to cost operator accordingly.

## Requirement

At high level, there are 2 requirements

1. Include current pod count in API response.
    1. ❓ Should this be single value of last datapoint just like current resource requests and limits? If yes, it can be quick to add pod count in existing current configuration section common to all recommendation terms.
    2. ❓ Should this be the min & max for each recommendation term? If yes, placement of this should be for each term. As the time window is different for different terms including custom terms (if any).

2. Recommend pod count in API response.
    1. ❓ Should this be a static value just like resource requests and limits we generate?
    2. ❓ Should this be a range value with min and max like HPA configuration?

⚠️ Apart from these, do we also need to consider if we have to generate HPA & VPA configuration like the samples shown above?

## Decisions

Following decisions are made

- HPA recommendations will be maintained separately outside the kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].config as separate configuration as it involve different set of attributes.
- Current pod count (actual) and its aggregate functions should be available for each term.
- Recommended pod count (whenever we have it) should be made available in kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].config.
- Restructure kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].config to maintain same structure as deployment yaml.

## API Response (v0.3)

Based on the decisions made in this ADR, please find the json response as shown below.

```jsonc
[
  {
    "cluster_name": "default",
    "experiment_name": "monitor_tfb_benchmark",
    "experiment_type": "container",
    "kubernetes_objects":
    [
      {
        "containers":
        [
          {
            "container_image_name": "kruize/tfb-qrh:1.13.2.F_et17",
            "container_name": "tfb-server",
            "recommendations":
            {
              "data":
              {
                "2025-11-17T10:31:51.000Z":
                {
                  "current":
                  {
                    "limits": {},
                    "requests": {}
                  },
                  "monitoring_end_time": "2025-11-17T10:31:51.000Z",
                  "recommendation_terms":
                  {
                    "short_term":
                    {
                      // New attribute : current; max and min replica of recommendation term
                      "current":
                      {
                        "replicas":
                        {
                          "avg": 2,
                          "max": 3,
                          "min": 1
                        }
                      },
                      "duration_in_hours": 24.0,
                      "monitoring_start_time": "2025-11-16T10:31:51.000Z",
                      "recommendation_engines":
                      {
                        "cost":
                        {
                          // Removed attribute : pods_count
                          "config":
                          {
                            "replicas": 4, // new attribute : replicas; recommended static value.
                            "resources": // New section
                            {
                              "limits": {},
                              "requests": {}
                            },
                            "env":
                            [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              }
                            ]
                          },
                          "variation":
                          {
                            // new attribute : replicas; variation w.r.t max replicas for each recommendation term.
                            "replicas": 1,
                            "resources": // New section
                            {
                              "limits": {},
                              "requests": {}
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        ],
        "name": "tfb-qrh-sample-0",
        "namespace": "default",
        "type": "deployment"
      }
    ],
    "version": "v2.0"
  }
]
```

## API Response (v0.4)

After addressing feedback from Kruize and ROS team, following is the json response.

```jsonc
[
  {
    "cluster_name": "default",
    "experiment_name": "monitor_tfb_benchmark",
    "experiment_type": "container",
    "kubernetes_objects":
    [
      {
        "containers":
        [
          {
            "container_image_name": "kruize/tfb-qrh:1.13.2.F_et17",
            "container_name": "tfb-server",
            "recommendations":
            {
              "data":
              {
                "2025-11-17T10:31:51.000Z":
                {
                  "current":
                  {
                    "replicas": 3, // new attribute : replicas; current value.
                    "resources": 
                    {
                      "limits": {}, // nested under resources
                      "requests": {} // nested under resources
                    }
                  },
                  "monitoring_end_time": "2025-11-17T10:31:51.000Z",
                  "recommendation_terms":
                  {
                    "short_term":
                    {
                      // New attribute : current; max and min replica of recommendation term
                      "metrics_info":
                      {
                        "pod_count":
                        {
                          "avg": 2,
                          "max": 3,
                          "min": 1
                        }
                      },
                      "duration_in_hours": 24.0,
                      "monitoring_start_time": "2025-11-16T10:31:51.000Z",
                      "recommendation_engines":
                      {
                        "cost":
                        {
                          // Removed attribute : pods_count
                          "config":
                          {
                            "replicas": 4, // new attribute : replicas; recommended static value.
                            "resources": 
                            {
                              "limits": {}, // nested under resources
                              "requests": {} // nested under resources
                            },
                            "env":
                            [
                              {
                                "name": "JDK_JAVA_OPTIONS",
                                "value": "-server -XX:+UseZGC -XX:MaxRAMPercentage=80 "
                              }
                            ]
                          },
                          "variation":
                          {
                            // new attribute : replicas; variation w.r.t max replicas for each recommendation term.
                            "replicas": 1,
                            "resources": // New section
                            {
                              "limits": {},
                              "requests": {}
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        ],
        "name": "tfb-qrh-sample-0",
        "namespace": "default",
        "type": "deployment"
      }
    ],
    "version": "v2.0"
  }
]
```

This include following changes since last version

- kubernetes_objects[n].containers[n].recommendations.data.[ts].current is modified to keep it inline with the deployment yaml.
- Included 'replicas' in current configuration kubernetes_objects[n].containers[n].recommendations.data.[ts].current
- renamed kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].current to kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].metrics_info
- renamed kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].current.replicas to kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].metrics_info.pod_count

## External Dependency

Our solution revolves around the metrics we currently collect. From these, we could compute the pods count using formulae mentioned above. However, it is advisable to get the metrics from Cost Operator. Till the time we get the metrics data available from cost operator, we will use computed value for pod count as a stop gap solution.

### Cost Management Queries

```promql
sum(kube_pod_container_status_ready{namespace="$NAMESPACE$", container="$CONTAINER_NAME$"})
avg_over_time(sum(kube_pod_container_status_ready{namespace="$NAMESPACE$", container="$CONTAINER_NAME$"})[15m:])
max_over_time(sum(kube_pod_container_status_ready{namespace="$NAMESPACE$", container="$CONTAINER_NAME$"})[15m:])
min_over_time(sum(kube_pod_container_status_ready{namespace="$NAMESPACE$", container="$CONTAINER_NAME$"})[15m:])
```

## Changes Required

### Kruize team

| Task Description | Dev Efforts | Owner |
| --- | --- | --- |
| Update performance profile to include queries for new metric 'podCount' | 0.5 pd | Saad |
| Open ticket with ROS/CostManagement to include queries for metric 'podCount' | 0.5 pd | Bhaktavatsal |
| Support new metric 'podCount' in updateResults endpoint | 1 pd | Saad |
| Update logic of computing replicas in this order 'replicas -> cpuUsage -> memory -> 0' | 0.5 pd | Saad |
| Create new model object to accommodate new json structure | 0.5 pd | Saad |
| Create new API /kruize/api/v1/recommendations | 0.5 pd | Saad |
| Open ticket with ROS team to consume new API endpoint | 0.5 pd | Bhaktavatsal |
| Open ticket with ROS team to capture and include new metric 'podCount' in updateResults API | 0.5 pd | Bhaktavatsal |
| Changes to Kruize tests to include tests for new API endpoint | 1 pd | Saad |

### Cost Management team

| Task Description | Dev Efforts |
| --- | --- |
| Include queries and start collecting and persist data for new metric 'podCount' | |

### ROS team

| Task Description | Dev Efforts |
| --- | --- |
| Consume new API endpoint /kruize/api/v1/recommendations | |
| Handle structural changes kubernetes_objects[n].containers[n].recommendations.data.[ts].current json of new API | |
| Handle new section kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].metrics_info | |
| Handle structural changes in kubernetes_objects[n].containers[n].recommendations.data.[ts].recommendation_terms.[term].config with fallback logic for backward compatibility | |
| Handle new metric 'podCount' from cost management and send to Kruize via updateResults API | |
| UI changes to show actual pod count, min & max for recommendation term | |

## Conclusion

Kruize will create new API endpoint for recommendations that gives response with schema as finalized in [API Response (v0.4)](URL "https://github.com/kruize/kruize-docs/blob/main/design/pod-count/pod-count-adr.md#api-response-v04")
