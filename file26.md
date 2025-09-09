# 字符串转bytes
def stringToBytes(_str):
    return bytes(_str, encoding="utf-8")
    # 或者 return str.encode(encoding="utf-8")


# bytes转字符串
def bytesToString(bs):
    return bytes.decode(bs, encoding="utf-8")


# 十六进制字符串转bytes
def hexStringToBytes(_str):
    _str = str.replace(" ", "")  # 去掉所有空格
    return bytes.fromhex(str)
    # 或者   from binascii import a2b_hex
    # return a2b_hex(_str)


# bytes转十六进制字符串
def bytesToHexString(bs):
    return bs.hex()
    # 2.
    # '%02X' % b 等同于 f"{b:02X}" 等同于 "{:02X}.format(b)"以16进制输出

    # return "".join(['%02X' % b for b in bs])

    # 3.
    # hex(10) 10进制转16进制 int("0x41",16) 16进制转10进制
    # hex(item)[2:] 去掉0x 如"0xAB" 去掉0x
    # zfill(2)返回指定长度的字符串，原字符串右对齐，前面填充0。
    # upper() 小写字母转为大写字母。

    # hex_str = ""
    # for item in bs:
    #     hex_str += str(hex(item))[2:].zfill(2).upper() + " "
    # return hex_str