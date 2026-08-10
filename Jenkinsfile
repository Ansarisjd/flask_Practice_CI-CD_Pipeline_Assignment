stage('MongoDB Connection Test') {
    steps {
        withCredentials([
            string(
                credentialsId: 'MONGO_URI',
                variable: 'MONGO_URI'
            )
        ]) {
            bat 'python -c "import ssl; print(ssl.OPENSSL_VERSION)"'
            bat 'python -c "import certifi; print(certifi.where())"'
            bat 'python -c "import pymongo; print(pymongo.version)"'
            bat 'python -c "import os; print(\"MONGO_URI exists:\", bool(os.environ.get(\"MONGO_URI\")))"'
        }
    }
}