from modules import cbpi
from thread import start_new_thread
import logging
import urllib, json, httplib, requests

DEBUG = False
ubidots_token = None
ubidots_label = None
drop_first = None

def log(s):
    if DEBUG:
        s = "IOT: " + s
        cbpi.app.logger.info(s)

def httpCon(url, path='', data_load='', meth='GET'):
    log("%s to URL %s - %s - %s" % (meth, url, path, data_load))
    try:
        data_load = eval(data_load)
        params = urllib.urlencode(data_load)
        conn = httplib.HTTPSConnection(url)
        path += "?" + params if params != "" else ""
        log("Path: %s" % path)
        conn.request(meth, path)
        response = conn.getresponse()
        try:
            json_result = response.read()
            json_result = json.loads(json_result)
            return json_result
        except Exception as e:
            return response
    except Exception as e:
        cbpi.app.logger.error("FAILED when contacting site: %s" % (url))
        cbpi.notify("IOT Error", "Check API and Channel ID.", type="danger", timeout=10000)
    return False

def httpJSON(url, path='', param='', data=''):
    log("URL %s - %s - %s, json %s" % (url, path, param, data))
    param = eval(param)
    params = urllib.urlencode(param)
    url += path
    url += "?" + params if params != "" else ""
    headers = {'content-type': 'application/json'}
    log("URL: %s, JSON: %s" % (url, data))
    try:
        response = requests.post(url, data=data, headers=headers)
        log("response %s" % response)
        return response
    except:
        return False

def ubidotsLabel():
    global ubidots_label
    ubidots_label = cbpi.get_config_parameter("ubidots_label", None)
    if ubidots_label is None:
        log("Init Ubidots Config Label ID")
        try:
            cbpi.add_config_parameter("ubidots_label", "", "text", "Ubidots API Label")
        except:
            cbpi.notify("Ubidots Error", "Unable to update config parameter", type="danger")

def ubidotsAPI():
    global ubidots_token
    ubidots_token = cbpi.get_config_parameter("ubidots_token", None)
    if ubidots_token is None:
        log("Init Ubidots Token")
        try:
            cbpi.add_config_parameter("ubidots_token", "", "text", "Ubidots API Token")
        except:
            cbpi.notify("Ubidots Error", "Unable to update config parameter", type="danger")

def Fillfield(jsonstr, key, value):
    data = ""
    if key in jsonstr:
        data = ", '%s':'%s'" % (key, value) if jsonstr[key] == "" else ""
    else:
        data = ", '%s':'%s'" % (key, value)
    return data

@cbpi.initalizer(order=9000)
def init(cbpi):
    cbpi.app.logger.info("Ubidots plugin Initialize")
    ubidotsAPI()
    ubidotsLabel()
    if ubidotsAPI is None:
        cbpi.notify("Ubidots Error", "Token key missing for Ubidots plugin, please update and reboot", type="danger")
    if ubidotsLabel is None:
        cbpi.notify("Ubidots Error", "Ubidots API label missing for Ubidots plugin, please update and reboot", type="danger")

def UbidotsUpdate(data):
    global ubidots_token
    global ubidots_label
    log("Ubidots update")
    url = "http://things.ubidots.com"
    if (ubidots_token == "" or ubidots_label == ""):
        log("Ubidots Token or label incorrect")
        cbpi.notify("Ubidots Error", "Please try to update config parameter and reboot.", type="danger")
        return False
    for count, (key, value) in enumerate(cbpi.cache["kettle"].iteritems(), 1):
        if value.target_temp is not None:
            data += ", \"target_temp_%s\":%s" % (count,value.target_temp)
    for count, (key, value) in enumerate(cbpi.cache["actors"].iteritems(), 1):
        if value.state is not None:
            data += ", \"actor_%s\":%s" % (value.name,value.state)
    data += "}"
    path = "/api/v1.6/devices/%s/" % ubidots_label
    param = "{'token':'%s'}" % ubidots_token
    result = httpJSON(url, path, param, data)

@cbpi.backgroundtask(key="ubidots_task", interval=60)
def ubidots_background_task(api):
    log("Ubidots background task")
    global drop_first
    if drop_first is None:
        drop_first = False
        return False
    cnt = 1
    dataT= ""
    dataU= "{"
    for key, value in cbpi.cache.get("sensors").iteritems():
        dataT += ", 'field%s':'%s'" % (cnt, value.instance.last_value)
        dataU += ", " if key >1 else ""
        dataU += "\"%s\":%s" % (value.name, value.instance.last_value)
        cnt += 1
    dataT += "}"
    log("Ubidots")
    UbidotsUpdate(dataU)
