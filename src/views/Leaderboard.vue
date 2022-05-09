<style>
	table {
		border-collapse: collapse;
		margin: 15px;
		box-sizing: border-box;
		font-size: 0.9em;
		font-family: sans-serif;
		box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
	}
	table thead tr {
		background: #1A73E8;
		color: #ffffff;
		text-align: left;
	}
	table th,
	table td {
		padding: 7px;
	}
	table tbody tr {
		border-bottom: 1px solid #dddddd;
		background: #fff;
	}
	table tbody tr:nth-of-type(even) {
		background: #f8f8f8;
	}
	table tbody tr:last-of-type {
		border-bottom: 2px solid #1A73E8;
	}
	table tbody tr:hover {
		background: rgba(179, 197, 229, 0.21);
	}
	@media only screen and (max-width: 768px) {
		table {
			width: calc(100% - 14px);
			margin: 7px;
		}
		table th,
		table td {
			padding: 4px;
		}
	}
</style>
<template>
	<table id="table">
		<thead>
			<tr>
		        <th>Position</th>
				<th>Holder</th>
				<th>Balance</th>
				<th>Opensea</th>
			</tr>
		</thead>
		<tbody>
		</tbody>
	</table>
</template>
<script type="text/javascript">
fetch('https://api.covalenthq.com/v1/1/tokens/0x93a796B1E846567Fe3577af7B7BB89F71680173a/token_holders/?quote-currency=USD&format=JSON&block-height=latest&page-size=50&key=ckey_c051e3b9081a474ab19214610fc')
	//replace api key with SPT accounts key
	.then(function (response) {
		return response.json();
	})
	.then(function (data) {
		appendData(data.data.items);
	})
	.catch(function (err) {
		console.log('error: ' + err);
	});
	
	function appendData(items) {
			var tbody = document.getElementById('table').getElementsByTagName('tbody')[0];
		for (var i = 1; i < items.length; i++) {
			var newRow = tbody.insertRow();
			var newCellPosition = newRow.insertCell();
			var newCellHolder = newRow.insertCell();
			var newCellBalance = newRow.insertCell();
			var newCellOpensea = newRow.insertCell();
			var newTextPosition = document.createTextNode(i);
			var newTextHolder = document.createTextNode(items[i].address);
			var newTextBalance = document.createTextNode(items[i].balance);
			var newTextOpensea = document.createTextNode(items[i].address);
			var url = "https://opensea.io/" + items[i].address;
			var element = document.createElement("a");
            element.setAttribute("href", url);
            element.innerHTML = "Link";
			newCellPosition.appendChild(newTextPosition);
			newCellHolder.appendChild(newTextHolder);
			newCellBalance.appendChild(newTextBalance);
			newCellOpensea.appendChild(element);
					}
}
function paginator(config) {
    // throw errors if insufficient parameters were given
    if (typeof config != "object")
        throw "Paginator was expecting a config object!";
    if (typeof config.get_rows != "function" && !(config.table instanceof Element))
        throw "Paginator was expecting a table or get_row function!";

    // get/set if things are disabled
    if (typeof config.disable == "undefined") {
        config.disable = false;
    }

    // get/make an element for storing the page numbers in
    var box;
    if (!(config.box instanceof Element)) {
        config.box = document.createElement("div");
    }
    box = config.box;

    // get/make function for getting table's rows
    if (typeof config.get_rows != "function") {
        config.get_rows = function () {
            var table = config.table
            var tbody = table.getElementsByTagName("tbody")[0]||table;

            // get all the possible rows for paging
            // exclude any rows that are just headers or empty
            children = tbody.children;
            var trs = [];
            for (var i=0;i<children.length;i++) {
                if (children[i].nodeType = "tr") {
                    if (children[i].getElementsByTagName("td").length > 0) {
                        trs.push(children[i]);
                    }
                }
            }

            return trs;
        }
    }
    var get_rows = config.get_rows;
    var trs = get_rows();

    // get/set rows per page
    if (typeof config.rows_per_page == "undefined") {
        var selects = box.getElementsByTagName("select");
        if (typeof selects != "undefined" && (selects.length > 0 && typeof selects[0].selectedIndex != "undefined")) {
            config.rows_per_page = selects[0].options[selects[0].selectedIndex].value;
        } else {
            config.rows_per_page = 10;
        }
    }
    var rows_per_page = config.rows_per_page;

    // get/set current page
    if (typeof config.page == "undefined") {
        config.page = 1;
    }
    var page = config.page;

    // get page count
    var pages = (rows_per_page > 0)? Math.ceil(trs.length / rows_per_page):1;

    // check that page and page count are sensible values
    if (pages < 1) {
        pages = 1;
    }
    if (page > pages) {
        page = pages;
    }
    if (page < 1) {
        page = 1;
    }
    config.page = page;
 
    // hide rows not on current page and show the rows that are
    for (var i=0;i<trs.length;i++) {
        if (typeof trs[i]["data-display"] == "undefined") {
            trs[i]["data-display"] = trs[i].style.display||"";
        }
        if (rows_per_page > 0) {
            if (i < page*rows_per_page && i >= (page-1)*rows_per_page) {
                trs[i].style.display = trs[i]["data-display"];
            } else {
                // Only hide if pagination is not disabled
                if (!config.disable) {
                    trs[i].style.display = "none";
                } else {
                    trs[i].style.display = trs[i]["data-display"];
                }
            }
        } else {
            trs[i].style.display = trs[i]["data-display"];
        }
    }

    // page button maker functions
    config.active_class = config.active_class||"active";
    if (typeof config.box_mode != "function" && config.box_mode != "list" && config.box_mode != "buttons") {
        config.box_mode = "button";
    }
    if (typeof config.box_mode == "function") {
        config.box_mode(config);
    } else {
        var make_button;
        if (config.box_mode == "list") {
            make_button = function (symbol, index, config, disabled, active) {
                var li = document.createElement("li");
                var a  = document.createElement("a");
                a.href = "#";
                a.innerHTML = symbol;
                a.addEventListener("click", function (event) {
                    event.preventDefault();
                    this.parentNode.click();
                    return false;
                }, false);
                li.appendChild(a);

                var classes = [];
                if (disabled) {
                    classes.push("disabled");
                }
                if (active) {
                    classes.push(config.active_class);
                }
                li.className = classes.join(" ");
                li.addEventListener("click", function () {
                    if (this.className.split(" ").indexOf("disabled") == -1) {
                        config.page = index;
                        paginator(config);
                    }
                }, false);
                return li;
            }
        } else {
            make_button = function (symbol, index, config, disabled, active) {
                var button = document.createElement("button");
                button.innerHTML = symbol;
                button.addEventListener("click", function (event) {
                    event.preventDefault();
                    if (this.disabled != true) {
                        config.page = index;
                        paginator(config);
                    }
                    return false;
                }, false);
                if (disabled) {
                    button.disabled = true;
                }
                if (active) {
                    button.className = config.active_class;
                }
                return button;
            }
        }

        // make page button collection
        var page_box = document.createElement(config.box_mode == "list"?"ul":"div");
        if (config.box_mode == "list") {
            page_box.className = "pagination";
        }

        var left = make_button("&laquo;", (page>1?page-1:1), config, (page == 1), false);
        page_box.appendChild(left);

        for (var i=1;i<=pages;i++) {
            var li = make_button(i, i, config, false, (page == i));
            page_box.appendChild(li);
        }

        var right = make_button("&raquo;", (pages>page?page+1:page), config, (page == pages), false);
        page_box.appendChild(right);
        if (box.childNodes.length) {
            while (box.childNodes.length > 1) {
                box.removeChild(box.childNodes[0]);
            }
            box.replaceChild(page_box, box.childNodes[0]);
        } else {
            box.appendChild(page_box);
        }
    }

    // make rows per page selector
    if (!(typeof config.page_options == "boolean" && !config.page_options)) {
        if (typeof config.page_options == "undefined") {
            config.page_options = [
                { value: 5,  text: '5'   },
                { value: 10, text: '10'  },
                { value: 20, text: '20'  },
                { value: 50, text: '50'  },
                { value: 100,text: '100' },
                { value: 0,  text: 'All' }
            ];
        }
        var options = config.page_options;
        var select = document.createElement("select");
        for (var i=0;i<options.length;i++) {
            var o = document.createElement("option");
            o.value = options[i].value;
            o.text = options[i].text;
            select.appendChild(o);
        }
        select.value = rows_per_page;
        select.addEventListener("change", function () {
            config.rows_per_page = this.value;
            paginator(config);
        }, false);
        box.appendChild(select);
    }

    // status message
    var stat = document.createElement("span");
    stat.innerHTML = "On page " + page + " of " + pages
        + ", showing rows " + (((page-1)*rows_per_page)+1)
        + " to " + (trs.length<page*rows_per_page||rows_per_page==0?trs.length:page*rows_per_page)
        + " of " + trs.length;
    box.appendChild(stat);

    // hide pagination if disabled
    if (config.disable) {
        if (typeof box["data-display"] == "undefined") {
            box["data-display"] = box.style.display||"";
        }
        box.style.display = "none";
    } else {
        if (box.style.display == "none") {
            box.style.display = box["data-display"]||"";
        }
    }

    // run tail function
    if (typeof config.tail_call == "function") {
        config.tail_call(config);
    }

    return box;
}  
            </script>