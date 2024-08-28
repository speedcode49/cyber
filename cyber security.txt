from PIL import Image

def genData(data):
    newd = []
    for i in data:
        newd.append(format(ord(i), "08b"))
    return newd

def modPix(pix, data):
    datalist = genData(data)
    lendata = len(datalist)
    imdata = iter(pix)

    for i in range(lendata):
        pix = [value for value in next(imdata)[:3] +
                                next(imdata)[:3] +
                                next(imdata)[:3]]

        for j in range(0, 8):
            if (datalist[i][j] == '0' and pix[j] % 2 != 0):
                pix[j] -= 1
            elif (datalist[i][j] == '1' and pix[j] % 2 == 0):
                if pix[j] != 0:
                    pix[j] -= 1
                else:
                    pix[j] += 1

        if (i == lendata - 1):
            if (pix[-1] % 2 == 0):
                if pix[-1] != 0:
                    pix[-1] -= 1
                else:
                    pix[-1] += 1
        else:
            if (pix[-1] % 2 != 0):
                pix[-1] -= 1

        pix = tuple(pix)
        yield pix[0:3]
        yield pix[3:6]
        yield pix[6:9]

def encode_enc(newimg, data):
    w = newimg.size[0]
    (x, y) = (0, 0)

    for pixel in modPix(newimg.getdata(), data):
        newimg.putpixel((x, y), pixel)
        if (x == w - 1):
            x = 0
            y += 1
        else:
            x += 1

def encode():
    img = input("Enter image name (with extension): ")
    try:
        image = Image.open(img, 'r')
    except FileNotFoundError:
        print(f"Error: File {img} not found.")
        return
    except IOError:
        print("Error: Cannot open image.")
        return

    data = input("Enter data to be encoded: ")
    if len(data) == 0:
        raise ValueError('Data is empty')

    password = input("Enter password: ")
    if len(password) == 0:
        raise ValueError('Password is empty')

    data += password

    # Convert image to RGBA format internally if not already
    if image.mode != 'RGBA':
        image = image.convert('RGBA')
    
    newimg = image.copy()
    encode_enc(newimg, data)

    new_img_name = input("Enter the name of new image (with extension): ")
    newimg.save(new_img_name)

def decode():
    img = input("Enter image name (with extension): ")
    try:
        image = Image.open(img, 'r')
    except FileNotFoundError:
        print(f"Error: File {img} not found.")
        return ""
    except IOError:
        print("Error: Cannot open image.")
        return ""

    password = input("Enter password: ")
    if len(password) == 0:
        raise ValueError('Password is empty')

    # Convert image to RGBA format internally if not already
    if image.mode != 'RGBA':
        image = image.convert('RGBA')

    data = ''
    imgdata = iter(image.getdata())

    while True:
        pixels = [value for value in next(imgdata)[:3] +
                                next(imgdata)[:3] +
                                next(imgdata)[:3]]

        binstr = ''

        for i in pixels[:8]:
            if i % 2 == 0:
                binstr += '0'
            else:
                binstr += '1'

        data += chr(int(binstr, 2))
        if pixels[-1] % 2 != 0:
            break

    if password in data:
        return data.replace(password, '')
    else:
        print("Incorrect password")
        return ""

def main():
    while True:
        try:
            a = int(input(":: Welcome ::\n1. Encode\n2. Decode\n3. Exit\n"))
            if a == 1:
                encode()
            elif a == 2:
                decoded_message = decode()
                if decoded_message:
                    print("Decoded message: " + decoded_message)
            elif a == 3:
                print("Exiting...")
                break
            else:
                print("Enter correct input")
        except ValueError:
            print("Invalid input, please enter a number.")

main()