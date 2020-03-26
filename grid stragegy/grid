var priceNow = 2.33;
var gridStatus = 0;
var gridLinePrice = [];
var gridLineStatus = [];
//var site = 11;
var gridId = [];
var spotInitAccount = null;
var spotLastAccount = null;
var futuresInitAccount = null;
var futuresLastAccount = null;
var spotInitStocks = 0;
var spotLastStocks = 0;
var spotInitBalance = 0;
var spotLastBalance = 0;
var spotInitProperty = 0;
var initProperty = spotInitProperty;
var gridProfit = 0;
var floatingProfit = 0;
var numberPerTime = 175;
var sumTradingTimes = 0;
var dayTradingTimes = 0;
var dayBeforeTimes = 0;
var strategyStartTime;
var strategyRunTime = 0;
var sumRunTime;
var initAvgPrice = 0;
var lastAvgPrice = 2.33;



function initGridPrice(price, site, gridInterval, gridNumber) {
    var gridLinePrice = [];
    if (site == 0) {
        gridLinePrice[0] = price;
    } else {
        gridLinePrice[0] = _N(price / Math.pow(1 + gridInterval / 100, site), 3);
    }
    for (var i = 1; i <= gridNumber + 1; i++) {
        gridLinePrice[i] = _N(gridLinePrice[i - 1] * (1 + gridInterval / 100), 3);
    }
    return gridLinePrice;
}

function initGridStatus(grid) {
    var gridLineStatus = [];
    for (var i = 0; i < grid.length; i++) {
        gridLineStatus[i] = 0;
    }
    return gridLineStatus;
}

function initGridId(grid) {
    var gridId = [];
    for (var i = 0; i < grid.length; i++) {
        gridLineStatus[i] = 0;
    }
    return gridId;
}

function gridSite(price, grid) {
    var s;
    if (price > grid[0] && price <= grid[grid.length - 1]) {
        for (var i = 0; i < grid.length - 1; i++) {
            if (price > grid[i] && price <= grid[i + 1]) {
                s = i;
                break;
            }
        }
        return s;
    }
    if (price <= grid[0]) {
        return -1;
    }
    if (price > grid[grid.length - 1]) {
        return grid.length;
    }
}

function CancelPendingOrders() {
    while (true) {
        var orders = exchange.GetOrders();
        for (var i = 0; i < orders.length; i++) {
            exchange.CancelOrder(orders[i].Id);
            Sleep(30);
        }
        if (orders.length === 0) {
            break;
        }
    }
}

function CancelBuyOrders() {
    var orders = exchange.GetOrders();
    for (var i = 0; i < orders.length; i++) {
        if (orders[i].Type == ORDER_TYPE_BUY) {
            exchange.CancelOrder(orders[i].Id);
        }
        Sleep(30);
    }
}

function adjustAccount(aType, number) {
    var Account;
    var Stocks;
    var depth;
    var price;
    var id;
    var order;
    switch (aType) {
        case 'buy':
            while (true) {
                Account = _C(exchange.GetAccount);
                Stocks = Account.Stocks;
                if (Stocks >= number) {
                    break;
                }
                depth = _C(exchange.GetDepth);
                //var price = depth.Asks[0].Price; //吃单
                price = depth.Bids[0].Price; //挂单
                id = exchange.Buy(price, Math.max(_N((number - Stocks) * 1.001 + 1, 1), 0.1));
                Sleep(5000);
                order = _C(exchange.GetOrder, id);
                if (order.Status == ORDER_STATE_PENDING) {
                    exchange.CancelOrder(id);
                }
            }
            break;
        case 'sell':
            while (true) {
                Account = _C(exchange.GetAccount);
                Stocks = Account.Stocks;
                if (Stocks <= number + 1) {
                    break;
                }
                depth = _C(exchange.GetDepth);
                //price = depth.Asks[0].Price; //挂单
                price = depth.Bids[0].Price; //吃单
                id = exchange.Sell(price, Math.max(_N(Stocks - number, 1), 0.1));
                Sleep(5000);
                order = _C(exchange.GetOrder, id);
                if (order.Status == ORDER_STATE_PENDING) {
                    exchange.CancelOrder(id);
                }
            }
            break;
        default:
            break;
    }
}

function onTick(exchange) {
    var tiker = _C(exchange.GetTicker);
    priceNow = tiker.Last;
    var nowTs = new Date().getTime();
    sumRunTime = _N(strategyRunTime + (nowTs - strategyStartTime) / (1000 * 60 * 60 * 24), 4);
    if ((nowTs + 1000 * 60 * 60 * 8) % (1000 * 60 * 60 * 24) > (1000 * 60 * 1) && (nowTs + 1000 * 60 * 60 * 8) % (1000 * 60 * 60 * 24) < (1000 * 60 * 2)) {
        dayBeforeTimes = dayTradingTimes;
        dayTradingTimes = 0;
        spotLastAccount = _C(exchange.GetAccount);
        if (spotLastAccount.Balance >= priceNow * 1.001 && spotLastAccount.Stocks < 1) {
            exchange.Buy(priceNow, 1);
        }
        Sleep(30 * 1000);
    }
    if (gridStatus == 0) {
        CancelPendingOrders();
        gridLinePrice = initGridPrice(priceNow, site, gridInterval, gridNumber);
        gridLineStatus = initGridStatus(gridLinePrice);
        gridId = initGridId(gridLinePrice);
        spotLastAccount = _C(exchange.GetAccount);
        spotLastStocks = _N(spotLastAccount.Stocks, 4);
        spotLastBalance = _N(spotLastAccount.Balance, 3);
        numberPerTime = _N((spotLastBalance + spotLastStocks * priceNow) * 0.98 / (((Math.pow((1 + gridInterval / 100), site) - (1 + gridInterval / 100)) / (gridInterval / 100)) * gridLinePrice[0] + (gridNumber - site) * priceNow), 0);
        if (numberPerTime == 0) {
            throw "资金不足，无法运行策略。@";
        }
        if (spotLastStocks < (gridNumber - site) * numberPerTime) {
            adjustAccount('buy', (gridNumber - site) * numberPerTime);
        } else {
            numberPerTime = _N(spotLastStocks / (gridNumber - site), 0);
        }
        Log("网格运行区间：", gridLinePrice[1], "-", gridLinePrice[gridNumber], "，网格每次交易量：", _N(numberPerTime, 0), "。@");
        Sleep(200);
        spotLastAccount = _C(exchange.GetAccount);
        spotLastStocks = _N(spotLastAccount.Stocks + spotLastAccount.FrozenStocks, 4);
        spotLastBalance = _N(spotLastAccount.Balance + spotLastAccount.FrozenBalance, 3);
        if (spotLastStocks != 0) {
            lastAvgPrice = _N((spotInitBalance + spotInitStocks * initAvgPrice - spotLastBalance) / spotLastStocks, 3);
        }
        for (var i = site; i < gridNumber; i++) {
            spotLastAccount = _C(exchange.GetAccount);
            if (spotLastAccount.Stocks >= numberPerTime) {
                gridId[i] = exchange.Sell(gridLinePrice[i + 1], numberPerTime);
            }
            if (gridId[i] != null && gridId[i] != 0) {
                gridLineStatus[i] = 2;
            }
            Sleep(100);
        }
        for (var j = site - 1; j >= 1; j--) {
            spotLastAccount = _C(exchange.GetAccount);
            if (spotLastAccount.Balance >= gridLinePrice[j] * numberPerTime * 1.001) {
                gridId[j] = exchange.Buy(gridLinePrice[j], _N(numberPerTime * 1.001, 4));
                if (gridId[j] != null && gridId[j] != 0) {
                    gridLineStatus[j] = 1;
                }
            }
            Sleep(100);
        }
        gridStatus = 1;
    }

    if (gridStatus == 1) {
        site = gridSite(priceNow, gridLinePrice);
        for (var k = Math.max(site - 2, 1); k < Math.min(site + 2, gridNumber); k++) {
            //for (var k = 1; k < 21; k++) {
            if (gridLineStatus[k] == 1) {
                var order1 = _C(exchange.GetOrder, gridId[k]);
                if (order1.Status == ORDER_STATE_CLOSED) {
                    gridLineStatus[k] = 0;
                    spotLastAccount = _C(exchange.GetAccount);
                    spotLastStocks = _N(spotLastAccount.Stocks + spotLastAccount.FrozenStocks, 4);
                    if (spotLastStocks != 0) {
                        lastAvgPrice = _N((lastAvgPrice * (spotLastStocks - order1.DealAmount * 0.999) + order1.AvgPrice * order1.DealAmount) / spotLastStocks, 3);
                    }
                    Log(gridLinePrice[k], "买入成功，", gridLinePrice[k + 1], "下单卖出。");
                    if (spotLastAccount.Stocks >= numberPerTime) {
                        gridId[k] = exchange.Sell(gridLinePrice[k + 1], numberPerTime);
                        Sleep(200);
                        if (gridId[k] != null && gridId[k] != 0) {
                            gridLineStatus[k] = 2;
                        }
                    }
                }
                if (order1.Status == ORDER_STATE_CANCELED) {
                    gridLineStatus[k] = 0;
                }
            }
            if (gridLineStatus[k] == 2) {
                var order2 = _C(exchange.GetOrder, gridId[k]);
                if (order2.Status == ORDER_STATE_CLOSED) {
                    gridLineStatus[k] = 0;
                    dayTradingTimes++;
                    sumTradingTimes++;
                    gridProfit = _N(gridProfit + (gridLinePrice[k + 1] * numberPerTime * 0.999 - gridLinePrice[k] * numberPerTime * 1.001), 3);
                    spotLastAccount = _C(exchange.GetAccount);
                    spotLastStocks = _N(spotLastAccount.Stocks + spotLastAccount.FrozenStocks, 4);
                    if (spotLastStocks != 0) {
                        lastAvgPrice = _N((lastAvgPrice * (spotLastStocks + order2.DealAmount) - order2.AvgPrice * order2.DealAmount * 0.999) / spotLastStocks, 3);
                    }
                    //floatingProfit = _N((spotLastAccount.Balance + spotLastAccount.FrozenBalance + (spotLastAccount.Stocks + spotLastAccount.FrozenStocks) * gridLinePrice[k + 1]) - spotInitProperty, 3);
                    Log(gridLinePrice[k + 1], "卖出成功，", gridLinePrice[k], "下单买回。今日卖出次数：", dayTradingTimes, "，昨日卖出次数：", dayBeforeTimes, "，累计卖出次数：", sumTradingTimes, "。@");
                    if (spotLastAccount.Balance >= gridLinePrice[k] * numberPerTime * 1.001) {
                        gridId[k] = exchange.Buy(gridLinePrice[k], _N(numberPerTime * 1.001, 4));
                        Sleep(200);
                        if (gridId[k] != null && gridId[k] != 0) {
                            gridLineStatus[k] = 1;
                        }
                    }
                }
                if (order2.Status == ORDER_STATE_CANCELED) {
                    gridLineStatus[k] = 0;
                }
            }
            if (gridLineStatus[k] == 0) {
                spotLastAccount = _C(exchange.GetAccount);
                if (priceNow < gridLinePrice[k + 1] && spotLastAccount.Stocks >= numberPerTime) {
                    gridId[k] = exchange.Sell(gridLinePrice[k + 1], numberPerTime);
                    Sleep(200);
                    if (gridId[k] != null && gridId[k] != 0) {
                        gridLineStatus[k] = 2;
                    }
                }
                if (gridLineStatus[k] == 0 && priceNow > gridLinePrice[k] && spotLastAccount.Balance >= gridLinePrice[k] * numberPerTime * 1.001) {
                    gridId[k] = exchange.Buy(gridLinePrice[k], _N(numberPerTime * 1.001, 4));
                    Sleep(200);
                    if (gridId[k] != null) {
                        gridLineStatus[k] = 1;
                    }
                }
            }
        }

        if (site > gridNumber) {
            initAvgPrice = lastAvgPrice;
            spotLastAccount = _C(exchange.GetAccount);
            spotInitStocks = _N(spotLastAccount.Stocks + spotLastAccount.FrozenStocks, 4);
            spotInitBalance = _N(spotLastAccount.Balance + spotLastAccount.FrozenBalance, 3);
            site = _N(gridNumber * 2 / 3, 0);
            gridStatus = 0;
        }

        spotLastAccount = _C(exchange.GetAccount);
        floatingProfit = _N((spotLastAccount.Balance + spotLastAccount.FrozenBalance + (spotLastAccount.Stocks + spotLastAccount.FrozenStocks) * priceNow) - spotInitProperty, 3);
        LogProfit(floatingProfit, '&');
        var table = {
            type: 'table',
            title: '策略最近启动于' + _D(strategyStartTime) + '已运行' + sumRunTime + '天 网格状态@' + _D(),
            cols: ['当前价格：', priceNow, '网格运行空间：', gridLinePrice[1] + '-' + gridLinePrice[gridNumber], '当前网格位置：', site],
            rows: [
                ['当前持仓成本：', lastAvgPrice, '当前资金数：', _N(spotLastAccount.Balance + spotLastAccount.FrozenBalance, 3), '当前持仓数：', _N(spotLastAccount.Stocks + spotLastAccount.FrozenStocks, 3)],
                ['初始资产合计：', spotInitProperty, '当前资产合计：', _N((spotLastAccount.Balance + spotLastAccount.FrozenBalance + (spotLastAccount.Stocks + spotLastAccount.FrozenStocks) * priceNow), 3), '账户盈亏：', floatingProfit],
                ['账户收益率（%）：', _N(floatingProfit / initProperty * 100, 2), '预计年化收益率（%）：', _N(floatingProfit * 365 / initProperty / sumRunTime * 100, 2), '当前运行网格数：', gridNumber],
                ['网格收益：', gridProfit, '网格收益率（%）：', _N(gridProfit / initProperty * 100, 2), '网格年化收益率（%）：', _N(gridProfit * 365 / initProperty / sumRunTime * 100, 2)],
                ['今日卖出次数：', dayTradingTimes, '昨日卖出次数：', dayBeforeTimes, '累计卖出次数：', sumTradingTimes]
            ]
        };
        LogStatus('`' + JSON.stringify(table) + '`');
    }
    Sleep(100);
}

function onerror() {
    Log("注意！机器人因错误已停止运行！@");
    //CancelBuyOrders();
    _G("gridStatus", gridStatus);
    _G("gridLinePrice", gridLinePrice);
    _G("gridLineStatus", gridLineStatus);
    _G("gridId", gridId);
    _G("numberPerTime", numberPerTime);
    _G("gridProfit", gridProfit);
    _G("dayTradingTimes", dayTradingTimes);
    _G("sumTradingTimes", sumTradingTimes);
    _G("dayBeforeTimes", dayBeforeTimes);
    _G("lastAvgPrice", lastAvgPrice);
    strategyRunTime = sumRunTime;
    _G("strategyRunTime", strategyRunTime);
    _G("initProperty", initProperty);
}

function onexit() {
    //CancelBuyOrders();
    _G("gridStatus", gridStatus);
    _G("gridLinePrice", gridLinePrice);
    _G("gridLineStatus", gridLineStatus);
    _G("gridId", gridId);
    _G("numberPerTime", numberPerTime);
    _G("gridProfit", gridProfit);
    _G("dayTradingTimes", dayTradingTimes);
    _G("sumTradingTimes", sumTradingTimes);
    _G("dayBeforeTimes", dayBeforeTimes);
    _G("lastAvgPrice", lastAvgPrice);
    strategyRunTime = sumRunTime;
    _G("strategyRunTime", strategyRunTime);
    _G("initProperty", initProperty);
}

function main() {
    strategyStartTime = new Date().getTime();
    if (_G("gridStatus") != null) {
        gridStatus = _G("gridStatus");
    }
    if (compel == true) {
        gridStatus = 0;
    }
    gridLinePrice = _G("gridLinePrice");
    gridLineStatus = _G("gridLineStatus");
    gridId = _G("gridId");
    if (_G("numberPerTime") != null) {
        numberPerTime = _G("numberPerTime");
    }
    if (_G("gridProfit") != null) {
        gridProfit = _G("gridProfit");
    }
    dayTradingTimes = _G("dayTradingTimes");
    sumTradingTimes = _G("sumTradingTimes");
    dayBeforeTimes = _G("dayBeforeTimes");
    if (_G("strategyRunTime") != null) {
        strategyRunTime = _G("strategyRunTime");
    }
    spotInitAccount = spotLastAccount = _C(exchange.GetAccount);
    spotInitStocks = _N(spotInitAccount.Stocks + spotInitAccount.FrozenStocks, 4);
    spotInitBalance = _N(spotInitAccount.Balance + spotInitAccount.FrozenBalance, 3);
    if (_G("lastAvgPrice") != null) {
        lastAvgPrice = _G("lastAvgPrice");
    } else {
        var tiker = _C(exchange.GetTicker);
        lastAvgPrice = tiker.Last;
    }
    initAvgPrice = lastAvgPrice;
    spotInitProperty = _N(spotInitBalance + spotInitStocks * initAvgPrice, 3);
    if (_G("initProperty") != null) {
        initProperty = _G("initProperty");
    } else {
        initProperty = _N(spotInitProperty, 3);
    }
    LogReset();
    Log('交易所:', exchange.GetName(), "。账户资金：", spotInitBalance, "USDT，币数：", spotInitStocks, "个；资产合计：", initProperty, "USDT。@");
    LogStatus("策略准备运行...");
    LogProfitReset();
    _CDelay(180);
    while (true) {
        onTick(exchange);
        Sleep(30000);
    }
}
