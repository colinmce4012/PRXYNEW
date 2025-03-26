
import http.server
import socketserver
import requests

PORT = 8888

class Proxy(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        url = self.path[1:]  # Remove leading '/'
        try:
            response = requests.get(url)
            self.send_response(response.status_code)
            for key, value in response.headers.items():
                self.send_header(key, value)
            self.end_headers()
            self.wfile.write(response.content)
        except Exception as e:
            self.send_error(500, f'Error: {str(e)}')

    def do_POST(self):
        url = self.path[1:]  # Remove leading '/'
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        try:
            response = requests.post(url, data=post_data, headers=self.headers)
            self.send_response(response.status_code)
            for key, value in response.headers.items():
                self.send_header(key, value)
            self.end_headers()
            self.wfile.write(response.content)
        except Exception as e:
            self.send_error(500, f'Error: {str(e)}')

with socketserver.TCPServer(("", PORT), Proxy) as httpd:
    print(f"Serving at port {PORT}")
    httpd.serve_forever()
