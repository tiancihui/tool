package main

import (
	"compress/gzip"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strings"

	"github.com/PuerkitoBio/goquery"
)

type Line struct {
	Year        string
	Name        string
	PersonNum   string
	PersonNumed string
	MaxScore    string
	MinScore    string
	AvgScore    string
}

func main() {
	selectd := map[string]int{
		"2021": 42,
		"2020": 43,
		"2019": 41,
		"2018": 41,
		"2017": 41,
	}

	AllData = make(map[string][]Line)

	for k, v := range selectd {
		for i := 1; i < v; i++ {
			postForm1(k, i)
		}

	}

	for k, v := range AllData {
		line := k
		y_19 := ""
		y_20 := ""
		y_18 := ""
		y_17 := ""
		y_21 := ""

		empty_17 := "2017,,,,,,"
		empty_18 := "2018,,,,,,"
		empty_19 := "2019,,,,,,"
		empty_20 := "2020,,,,,,"
		empty_21 := "2021,,,,,,"

		for _, item := range v {
			if item.Year == "2019" {
				y_19 = item.Year + "," + item.Name + "," + item.PersonNum + "," + item.PersonNumed + "," + item.MaxScore + "," + item.MinScore + "," + item.AvgScore
			}

			if item.Year == "2020" {
				y_20 = item.Year + "," + item.Name + "," + item.PersonNum + "," + item.PersonNumed + "," + item.MaxScore + "," + item.MinScore + "," + item.AvgScore
			}

			if item.Year == "2021" {
				y_21 = item.Year + "," + item.Name + "," + item.PersonNum + "," + item.PersonNumed + "," + item.MaxScore + "," + item.MinScore + "," + item.AvgScore
			}

			if item.Year == "2017" {
				y_17 = item.Year + "," + item.Name + "," + item.PersonNum + "," + item.PersonNumed + "," + item.MaxScore + "," + item.MinScore + "," + item.AvgScore
			}

			if item.Year == "2018" {
				y_18 = item.Year + "," + item.Name + "," + item.PersonNum + "," + item.PersonNumed + "," + item.MaxScore + "," + item.MinScore + "," + item.AvgScore
			}
		}

		if len(y_19) == 0 {
			y_19 = empty_19
		}

		if len(y_20) == 0 {
			y_20 = empty_20
		}

		if len(y_21) == 0 {
			y_21 = empty_21
		}

		if len(y_17) == 0 {
			y_17 = empty_17
		}

		if len(y_18) == 0 {
			y_18 = empty_18
		}

		fmt.Println(line, ",", y_21, ",", y_20, ",", y_19, ",", y_18, ",", y_17)
	}

}

var AllData map[string][]Line

//以PostForm的方式发送body为键值对的post请求
func postForm1(year string, page int) {
	client := &http.Client{}
	data := urlValues(year, page)
	req, err := http.NewRequest("POST", "http://www.haeea.cn/HEAODataCenter_XCZX/PagePZQuery/ShowPZLQTJ.aspx/QueryInfo", strings.NewReader(data))

	req.Header.Add("Host", "www.haeea.cn")
	req.Header.Add("Connection", "keep-alive")
	req.Header.Add("Content-Length", "82")
	//req.Header.Add("Accept", "application/json, text/javascript, */*; q=0.01")
	req.Header.Add("Accept", "application/json")
	req.Header.Add("X-Requested-With", "XMLHttpRequest")
	req.Header.Add("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36")
	req.Header.Add("Content-Type", "application/json; charset=UTF-8")
	req.Header.Add("Origin", "http://www.haeea.cn")
	req.Header.Add("Referer", "http://www.haeea.cn/HEAODataCenter_XCZX/PagePZQuery/ShowPZLQTJ.aspx")
	req.Header.Add("Accept-Encoding", "gzip, deflate")
	req.Header.Add("Accept-Language", "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7")

	resp, err := client.Do(req)

	defer func() {
		resp.Body.Close()
	}()

	body := resp.Body
	if resp.Header.Get("Content-Encoding") == "gzip" {
		body, err = gzip.NewReader(resp.Body)
		if err != nil {
			fmt.Println("http resp unzip is failed,err: ", err)
		}
	}
	data_r, err := ioutil.ReadAll(body)

	//body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println(" post err=", err)
	}

	var r_data RData
	json.Unmarshal(data_r, &r_data)

	var headings, row []string
	var rows [][]string
	doc, err := goquery.NewDocumentFromReader(strings.NewReader(r_data.D.ShowInfo))
	if err != nil {
		fmt.Println("No url found")
		log.Fatal(err)
	}

	// Find each table
	doc.Find("table").Each(func(index int, tablehtml *goquery.Selection) {
		tablehtml.Find("tr").Each(func(indextr int, rowhtml *goquery.Selection) {
			rowhtml.Find("th").Each(func(indexth int, tableheading *goquery.Selection) {
				headings = append(headings, tableheading.Text())
			})
			rowhtml.Find("td").Each(func(indexth int, tablecell *goquery.Selection) {
				row = append(row, tablecell.Text())
			})
			rows = append(rows, row)
			row = nil
		})
	})
	//fmt.Println("####### headings = ", len(headings), headings)
	for _, item := range rows {
		//fmt.Println(len(item))
		if len(item) == 7 {
			per := Line{year, item[1], item[2], item[3], item[4], item[5], item[6]}
			if sub_item, ok := AllData[item[0]]; ok {
				sub_item = append(sub_item, per)
				AllData[item[0]] = sub_item
			} else {
				sub_item = append(sub_item, per)
				AllData[item[0]] = sub_item
			}
			//fmt.Println(item[0], item[1], item[2], item[3], item[4], item[5], item[6])
		}

		/*
			for _, item_sub := range item {
				fmt.Println(item_sub)
			}
		*/
	}

	//fmt.Println(r_data.D.ShowInfo)
}

//获取键值对的body
func urlValues(year string, page int) string {
	//方式1
	//方式2
	mapData := make(map[string]interface{})
	mapData["l_T"] = "2"
	//mapData["l_Y"] = "2021"
	mapData["l_Y"] = year
	mapData["l_K"] = "文科"
	mapData["l_P"] = "本科第二批"
	mapData["l_QT"] = ""
	//mapData["l_PI"] = 5
	mapData["l_PI"] = page

	b, _ := json.Marshal(mapData)

	return string(b)
}

type RData struct {
	D struct {
		ShowInfo   string `json:"ShowInfo"`
		LX         string `json:"LX"`
		TotalCount int    `json:"TotalCount"`
		PageSize   int    `json:"PageSize"`
	} `json:"d"`
}
