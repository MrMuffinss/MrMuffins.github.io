#include <iostream>
#include <string>
#include <boost/asio.hpp>

using namespace std;
using namespace boost::asio;

class Server {
public:
    Server(io_service& ios, int port) : acceptor(ios, ip::tcp::endpoint(ip::tcp::v4(), port)),
                                         socket(ios) {
        startAccept();
    }

private:
    void startAccept() {
        acceptor.async_accept(socket, [this](const boost::system::error_code& ec) {
            if (!ec) {
                handleRequest();
            }
            startAccept();
        });
    }

    void handleRequest() {
        boost::system::error_code ec;
        streambuf request;
        read_until(socket, request, "\r\n\r\n", ec);

        if (!ec) {
            istream request_stream(&request);
            string header;
            getline(request_stream, header);

            // Simple check for a GET request
            if (header.find("GET") == 0) {
                sendResponse();
            }
        }
    }

    void sendResponse() {
        string response = "HTTP/1.1 200 OK\r\nContent-Length: 13\r\n\r\nHello, World!";
        write(socket, buffer(response));
    }

    ip::tcp::acceptor acceptor;
    ip::tcp::socket socket;
};

int main() {
    try {
        io_service ios;
        Server server(ios, 8080);
        ios.run();
    } catch (exception& e) {
        cerr << "Exception: " << e.what() << endl;
    }

    return 0;
}
