pipeline {
    stages {
        stage('Build') {
            agent { docker 'golang' }
            steps {
                echo 'Building..'

                export GOPATH=$PWD/gopath
                export PATH=$PWD/gopath/bin:$PATH

                assets=$PWD/assets

                version="0.0.0-dev"
                if [ -d version ]; then
                  version="$(cat version/version)"
                fi

                pushd booklit
                  for os in linux darwin windows; do
                    ext=""
                    if [ "$os" = "windows" ]; then
                      ext=".exe"
                    fi

                    echo "building $os binary..."
                    GOOS=$os go build \
                      --ldflags "-X github.com/vito/booklit.Version=${version}" \
                      -o $assets/booklit_${os}_amd64${ext} \
                      github.com/vito/booklit/cmd/booklit
                  done
                popd

                echo "printing version..."
                ./assets/booklit_$(go env GOOS)_$(go env GOARCH) --version
            }
        }
        stage('Test') {
            agent { docker 'golang' }
            steps {
                echo 'Testing..'

                export GOPATH=$PWD/gopath
                export PATH=$PWD/gopath/bin:$PATH

                cd booklit

                echo "installing ginkgo..."
                go install github.com/onsi/ginkgo/ginkgo

                function emit_coveralls() {
                  if [ -n "$COVERALLS_TOKEN" ]; then
                    echo "emitting code coverage..."
                    go get github.com/mattn/goveralls
                    goveralls -service concourse \
                      -coverprofile <(find . -name '*.coverprofile' | xargs cat | grep -v 'booklit.peg.go\|render/bindata.go')
                  fi
                }

                trap emit_coveralls EXIT

                echo "running tests..."
                ./scripts/test -p "$@"
            }
        }
    }
}
